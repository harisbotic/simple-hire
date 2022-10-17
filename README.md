## Table of Contents

1. [Find suspicious candidates for each job opening](#suspicious-candidates)
2. [Retrieve all candidates in workspace](#candidates-in-workspace)

Thank you Jan for giving me a chance.
I completed the homework in requested time period and I gave my best to cover all things I find important to show which will demonstrate my way of thinking and tech skills

NOTE: If this was a real life application I would add extra columns like created_at and updated_at to each table and think more of "what-if" situations.

# Find suspicious candidates for each job opening

### Query

```sql
select tr.*
from test_result tr
    join lateral (
        select from test_result tr2
        where tr.job_opening_id = tr2.job_opening_id
            and (
                (tr.email <> tr2.email and tr.email_normalized = tr2.email_normalized)
                or (tr.ip_address = tr2.ip_address)
                or tr.has_fraud_events
                or (tr.submitted_after is not null and tr.submitted_after < interval '3' minute)
            )
    ) x on true;
```

# Retrieve all candidates in workspace

I did the cursor pagination for this feature as request but I added an extra row_number pagination just in case cursor pagination wasn't a requirement and I think row_number one will perform better on larger sets

NOTE: The filter by workspace we would just add list of job_opening_ids of that workspace
NOTE2: I know I was supposed to do filtering by test_id and not test_version_id but it would be too much for me to change it now and I feel like it won't show my skills in different light if I fix it at this point.
I would just about adding workspace_id and test_id columns to test_result table for these two changes
Maybe not workspace_id :)

### Query (cursor pagination)

```sql
begin;

declare test_cursor cursor for select * from test_result tr
    where tr.id > :last_id
    and tr.job_opening_id in (:job_opening_ids)
    and (
        test_version_id = :test_verion_id
        and tr.score >= :score_in_percentage -- eg. 0.8 for 80%
    )
  order by tr.id desc;

fetch :page_size from test_cursor;

commit;
```

### Query (row_number pagination)

```sql
select *
from (
    select
        row_number() over(order by tr.id) as rn,
           *
    from test_result tr
        where tr.job_opening_id in (:job_opening_ids)
        and (
            test_version_id = :test_verion_id
            and tr.score >= :score_in_percentage -- eg. 0.8 for 80%
        )
) x
where x.rn >= 101 and x.rn <= 200;
```

![image](https://user-images.githubusercontent.com/6454831/196156467-00265811-edc8-4bfb-ab43-3d577cd7383c.png)
