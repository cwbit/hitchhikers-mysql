# Slug Generator

The following set of scripts are useful in initial slug generation in a database. These were created as part of a client project - hopefully you find them useful, too.

### Initial generation
Create a slug from a combination of id + name. e.g. `1300-test-slug`.  

- Replace `schema.table` with your actual settings.
- Change `ROLLBACK` to `COMMIT` once you're comfortable with how things look.

```mysql
-- INITIAL SLUG GENERATION
BEGIN;
UPDATE `schema`.`table` SET -- initial conversion, don't run if you've already modified the slugs manually
    slug = concat(id, ' ',lower(name))
WHERE id <>123123123; -- satisfy 'safeupdate' whining :)
ROLLBACK;
```

### Slug Formatting
fix the formatting from existing or just-generated slugs. replace **`schema.table`** with your actual table name.

- Replace `schema.table` with your actual settings.
- Change `ROLLBACK` to `COMMIT` once you're comfortable with how things look.

```mysql
-- FIX SLUG FORMATTING
BEGIN;
UPDATE `schema`.`table` SET
	slug = lower(slug),				-- convert to lowercase
	slug = replace(slug, '-', 	' '), -- remove existing dashes
    slug = replace(slug, '.', 	' '), -- remove period
	slug = replace(slug, ',', 	' '), -- remove comma
	slug = replace(slug, ':', 	''), -- remove colon
	slug = replace(slug, ';', 	''), -- remove semi-colon
	slug = replace(slug, '?', 	''), -- remove question mark
    slug = replace(slug, '\'', 	''), -- remove quotes
	slug = replace(slug, 'â€™', ''), -- remove encoded quotes
	slug = replace(slug, '"', ''), -- remove encoded quotes
	slug = replace(slug, '"', ''), -- remove encoded quotes 
	slug = replace(slug, '"', ''), -- remove encoded quotes
	slug = replace(slug, '"', ''), -- remove encoded quotes
	slug = replace(slug, '!', 	''), -- remove bang
	slug = replace(slug, '(', 	''), -- remove paren
	slug = replace(slug, ')', 	''), -- remove paren
	slug = replace(slug, '|', 	''), -- remove pipe
    slug = trim(slug),				-- trim spaces
    slug = replace(slug, ' ', 	'-'), -- replace spaces
    slug = replace(slug, '--', 	'-') -- replace double dashed
WHERE id <>123123123;
UPDATE `schema`.`table` SET
    slug = replace(slug, '--', '-')
WHERE id <>123123123;
ROLLBACK;
```

### Find and Fix Duplicates
If you're not using an ID as part of your slug you may run into duplicate slug issues. These ugly little numbers will help solve that problem.

- Replace `schema.table` with your actual settings.
- Change `ROLLBACK` to `COMMIT` once you're comfortable with how things look.

```mysql
-- FIND DUPLICATES
select id as cid, name, slug, description from `schema`.`table` a where exists (select 'x' from `schema`.`table` b where b.slug = a.slug and b.id <> a.id) order by slug;

-- UPDATE DUPLICATES
BEGIN;
UPDATE `schema`.`table` set
	slug = concat(slug, '-', description) 
	where `schema`.`table`.id in (SELECT cid FROM (select id as cid from `schema`.`table` a where exists (select 'x' from `schema`.`table` b where b.slug = a.slug and b.id <> a.id)) as C)
	AND `schema`.`table`.id <> 1231313123;
ROLLBACK;
```