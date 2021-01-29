# Sequence_Reset procedure for entire schema@Postgresql after schema refresh:

**Note:**  Number(18) in following string needs to be change as per your schema_name length "left(substr(column_default,18),-12)"

Below function will reset all the sequnces in a schema to resceptive column max value +1.

create or replace function schema_name.seq_reset(p_schemaname text)
RETURNS text AS $$
declare
rec_curs RECORD;
v_val integer;
v_sql_1 TEXT DEFAULT '';
v_sql_2 TEXT DEFAULT '';
v_sql_3 TEXT DEFAULT '';
v_maxid integer;
seq_cursor cursor for select table_schema,table_name,column_name,left(substr(column_default,18),-12) sequence_name from information_schema.columns where table_schema=p_schemaname and column_default like 'nextval%' ;
begin
open seq_cursor;
loop
fetch seq_cursor into rec_curs;
exit when not found;
if rec_curs.table_name is not null then
v_sql_1 := 'alter sequence '||rec_curs.table_schema||'.'||rec_curs.sequence_name|| ' restart with ';
v_sql_2 := 'select coalesce(max('||rec_curs.column_name||'),0) +1 from '||rec_curs.table_schema||'.'||rec_curs.table_name;
--v_sql_3 is just for checking which statements executed through v_sql_1
v_sql_3 := v_sql_3||E'\n'||'alter sequence '||rec_curs.table_schema||'.'||rec_curs.sequence_name|| ' restart with ';
execute v_sql_2 into v_maxid;
execute v_sql_1 ||v_maxid;
end if;
end loop;
close seq_cursor;
return v_sql_3;
end; $$
LANGUAGE plpgsql;


### Calling the Function:

select schema_name.seq_reset('schemaname'); 
