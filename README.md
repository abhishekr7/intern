# intern

Angel Crunchbase coelesced

select distinct company_name, location, industries from(
select ac.company_name, array_join(ac.locations,', ') as location, array_join(ac.markets,', ') as industries
from growthpal.angelco_company_overview ac
where cardinality(urls) >= 1 and (url_extract_host(urls[1]) not in
(select regexp_replace(website, '(.*?)/?', '$1') as site
from growthpal.crunchbase_partitioned
where website <> ''
intersect 
select url_extract_host(urls[1]) as site
from growthpal.angelco_company_overview
where contains(markets, 'Healthcare') and urls is not null))
union all
select name as company_name, location, industries
from growthpal.crunchbase_partitioned
where name <> '' and regexp_replace(website, '(.*?)/?', '$1') in
(select regexp_replace(website, '(.*?)/?', '$1') as site
from growthpal.crunchbase_partitioned
where website <> ''
union 
select url_extract_host(urls[1]) as site
from growthpal.angelco_company_overview
where contains(markets, 'Healthcare') and urls is not null))
where location like '%Pune%' or location like '%Mumbai%'


Sector Expansion

SELECT coalesce(name,'') || coalesce(company_name,'') as company_name, cast(replace(monthly_visits,',','') as integer) as monthly_visits, location
FROM growthpal.crunchbase_partitioned 
WHERE location like '%India%' and (lower(industries) like '%b2b%' or lower(industries) like 'health%' or lower(industries) like 'medic%' or lower(industries) like 'bio%' or lower(industries) like 'hosp%' or lower(industries) like 'diag%') and (location like '%Pune%' or location like '%Mumbai%') and (last_funding_type <> '') and monthly_visits <> ''
   
Funding Amount

SELECT DISTINCT company_name, funding, industries
FROM ( SELECT company_name, CAST(REGEXP_REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(total_funding_amount,'$',''),'₹',''),'K','000'),'M','000000'),'B','000000000'),'(\d+)\.(\d+?)0','$1$2') AS bigint) as funding, location, industries
FROM growthpal.crunchbase_partitioned
WHERE company_name <> '' and total_funding_amount <> '' and (location LIKE 'Pune%' or location LIKE 'Mumbai%') and total_funding_amount LIKE '%₹%')
WHERE lower(industries) like '%b2b%' or lower(industries) like 'health%' or lower(industries) like 'medic%' or lower(industries) like 'bio%' or lower(industries) like 'hosp%' or lower(industries) like 'diag%'
ORDER BY funding desc

Funding Rounds

SELECT DISTINCT * FROM( 
SELECT coalesce(name,'') || coalesce(company_name,'') as company_name, industries, location, last_funding_type
FROM growthpal.crunchbase_partitioned
WHERE location like '%India%' and (lower(industries) like '%b2b%' or lower(industries) like 'health%' or lower(industries) like 'medic%' or lower(industries) like 'bio%' or lower(industries) like 'hosp%' or lower(industries) like 'diag%') and (location like '%Pune%' or location like '%Mumbai%') and (last_funding_type <> '')
ORDER BY
  CASE 
    WHEN last_funding_type = 'Angel' THEN 0
    WHEN last_funding_type = 'Pre-Seed' THEN 1
    WHEN last_funding_type = 'Seed' THEN 2
    WHEN last_funding_type = 'Venture - Series Unknown' THEN 3
    WHEN last_funding_type = 'Series A' THEN 4
    WHEN last_funding_type = 'Series B' THEN 5
    WHEN last_funding_type = 'Series C' THEN 6
    WHEN last_funding_type = 'Series D' THEN 7 
    WHEN last_funding_type = 'Equity Crowdfunding' THEN 8
    WHEN last_funding_type = 'Product Crowdfunding' THEN 9
    WHEN last_funding_type = 'Private Equity' THEN 10
    WHEN last_funding_type = 'Convertible Note' THEN 11
    WHEN last_funding_type = 'Debt Financing' THEN 12
    WHEN last_funding_type = 'Secondary Market' THEN 13
    WHEN last_funding_type = 'Grant' THEN 14
    WHEN last_funding_type = 'Corporate Round' THEN 15
    WHEN last_funding_type = 'Initial coin offering (ICO)' THEN 16
    WHEN last_funding_type = 'Post-IPO Equity' THEN 17
    WHEN last_funding_type = 'Post-IPO Debt' THEN 18
    WHEN last_funding_type = 'Post-IPO Secondary' THEN 19
    WHEN last_funding_type = 'Non-Equity Assistance' THEN 20
    WHEN last_funding_type = 'Funding Round' THEN 21
    ELSE 99
  END DESC
)

Crunchbase acquisitions

SELECT DISTINCT coalesce(name,'') || coalesce(company_name,'') as company_name, industries, number_of_acquisitions
FROM growthpal.crunchbase_partitioned
WHERE location like '%India%' and (lower(industries) like '%b2b%' or lower(industries) like 'health%' or lower(industries) like 'medic%' or lower(industries) like 'bio%' or lower(industries) like 'hosp%' or lower(industries) like 'diag%') and (location like '%Pune%' or location like '%Mumbai%') and (number_of_acquisitions <> '')
ORDER BY number_of_acquisitions DESC
