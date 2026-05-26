# Metabase Demo Setup

This directory runs local Metabase on port `3001` and connects it to the public
demo Redshift cluster created by Terraform.

## Start Metabase

```bash
cp .env.example .env
# Edit .env and set REDSHIFT_PASSWORD to the same value used in terraform.tfvars.
docker compose up -d
```

## Register Redshift

```bash
./setup-redshift-database.sh
```

The script creates the first local Metabase admin user when needed, discovers
the Redshift host from Terraform output or AWS, and registers a Redshift
database connection scoped to the `nyc_taxi` schema.

## Import Dashboards

```bash
./import-demo-dashboards.py
```

The importer creates or updates the `Nash DataOps Demo` collection and imports
the dashboard definitions from `dashboards/nash-dataops-demo.json`.

## Useful SQL

```sql
select count(*) as trips
from nyc_taxi.fact_fhvhv_trips;

select d.full_date as pickup_date, count(*) as trips
from nyc_taxi.fact_fhvhv_trips f
join nyc_taxi.dim_date d
  on f.pickup_date_key = d.date_key
group by 1
order by 1;

select coalesce(z.borough, 'Unknown') as pickup_borough, count(*) as trips
from nyc_taxi.fact_fhvhv_trips f
left join nyc_taxi.dim_zone z
  on f.pu_location_id = z.location_id
group by 1
order by trips desc;
```
