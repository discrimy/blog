Title: Next/Previous model in Django ORM
Date: 2024-08-19

## Task
I had a task: there is a table of posts `Post` with dynamic ordering based on user's input (newest first, by views count, etc.). I need to add button `Next` and `Previous` button on each posts' page. Ordering must be preserved between list view and serfing between posts.


## Solution
Preserving ordering between pages of list of posts and particular posts is trivial: just preserve query param of ordering in links. 
```python
# Link to list of pages
"/posts/?order=newest"
# Link to post
"/posts/123/?order=newest"
```

Getting next and previous models is trickier.

### Return to ~~monkey~~ SQL
It's good to solve a difficult problem in Django ORM by solving it in SQL before. Google leads me to the solution with using window functions. I won't explain what window functions are and how to use them (there are plenty of posts in the Internet and there're a way better at this). The part is needed for me are functions `Lag` and `Lead`. 
IDs are enough for me (I don't actually need next/prev models themselves, just IDs to build links).

```sql
-- I use PostgreSQL btw
SELECT
    *,
    LAG(id) OVER (ORDER BY dt_created DESC) as prev_id,
    LEAD(id) OVER (ORDER BY dt_created DESC) as next_id
FROM posts
WHERE 
    name ilike "%experiment%"
    AND id = 123
ORDER BY dt_created DESC
```

But there's a problem: `id = 123` clause removes all other rows from the result, and `LEAD` and `LAG` window functions returns nothing. The solution is to query next/prev ids in separate query and attach it later.

```sql
WITH "cte" AS (
    SELECT 
        id, 
        LAG(id) OVER (ORDER BY dt_created DESC) as prev_id,
        LEAD(id) OVER (ORDER BY dt_created DESC) as next_id
    FROM posts
    WHERE name ILIKE "%experiment%"
)
SELECT 
    cte.prev_id as prev_id,
    cte.next_id as next_id
FROM posts
INNER JOIN cte ON posts.id = cte.id
WHERE
    name ILIKE "%experiment%"
    AND posts.id = 123
```

Great, it returns the result, so we need to migrate the solution to Django ORM.

### Django ORM way
We used `CTE` to write a SQL query, so we need them in Django ORM too. Hopefully there'is a package [`django-cte`](https://github.com/dimagi/django-cte) to help us! Also I want solution to be composeable with the rest of QuerySet's methods, so it will be a method too.

```python
import django_cte

class PostQuerySet(django_cte.CTEQuerySet["Post"]):
    def with_prev_next_ids(self) -> Self:
        queryset_cte = self.annotate(
            prev_id=Window(expression=Lag("id"), order_by=self.query.order_by),
            next_id=Window(expression=Lead("id"), order_by=self.query.order_by),
        ).order_by()  # empty order_by() removes ordering because it's obsolete in CTE query part
        cte = django_cte.With(queryset_cte.values("id", "prev_id", "next_id"))  # select ids only
        queryset = (
            cte.join(self, id=cte.col.id)  # `INNER JOIN` part
            .with_cte(cte)  # `WITH "cte" (...)` part
            .annotate(
                prev_id=cte.col.prev_id,
                next_id=cte.col.next_id,
            )
        )
        return queryset

# Later in your code

queryset = Posts.objects.all()
# Apply ordering first
queryset = queryset.order_by("-dt_created")
# Attach next/prev ids later
queryset = queryset.with_prev_next_ids()
# (Optional) Get 1 object only, ignore if you need list of objects with ids
post = queryset.get(id=123)
```

Next/Prev button work now, everyone is happy.

## Optimization
Next/Prev ID calculation can be a hard task for DB and leads to slow query. But there is a trick: usually (90%) post views do not use these buttons, so it would be better to postpone calculation of next/prev IDs from post's page loading to click on the buttons.

- User loads page with post, both buttons leads to `/posts/<next/prev>/?<query>`
- User clicks a button
- Server performs actual filtering and ordering and gets next/prev ID
- Server replies with redirect to next/prev post using retrieved ID

Post page is fast as it was before, next/prev buttons works. Everyone is more happier. Well, there is a downside: you cannot disable next/prev button when there's no next or prev post because you don't you it on post load stage. 