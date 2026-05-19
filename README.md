CREATE OR REPLACE TEMP TABLE EXPORT_3_ENTITY_RELATIONSHIP_MAPPING AS
WITH pk_count AS (
    SELECT
        database_name,
        schema_name,
        table_name,
        COUNT(*) AS total_pk_columns
    FROM TMP_PK_SCOPE
    GROUP BY
        database_name,
        schema_name,
        table_name
),
matches AS (
    SELECT
        pk.database_name AS parent_database,
        pk.schema_name   AS parent_schema,
        pk.table_name    AS parent_table,
        pk.column_name   AS parent_column,
        pk.key_sequence  AS key_sequence,

        c.table_catalog AS child_database,
        c.table_schema  AS child_schema,
        c.table_name    AS child_table,
        c.column_name   AS child_column,
        c.data_type     AS child_data_type
    FROM TMP_PK_SCOPE pk
    INNER JOIN TMP_ALL_COLUMNS_SCOPE c
        ON UPPER(pk.column_name) = UPPER(c.column_name)

    
       AND (
              UPPER(pk.database_name) <> UPPER(c.table_catalog)
           OR UPPER(pk.schema_name)   <> UPPER(c.table_schema)
           OR UPPER(pk.table_name)    <> UPPER(c.table_name)
       )
),
relationship_group AS (
    SELECT
        m.parent_database,
        m.parent_schema,
        m.parent_table,
        m.child_database,
        m.child_schema,
        m.child_table,

        COUNT(DISTINCT m.parent_column) AS matched_pk_columns,
        MAX(pc.total_pk_columns) AS parent_total_pk_columns,

        LISTAGG(
            m.child_schema || '.' || m.child_table || '.' || m.child_column
            || ' = ' ||
            m.parent_schema || '.' || m.parent_table || '.' || m.parent_column,
            ' AND '
        ) WITHIN GROUP (ORDER BY m.key_sequence) AS join_condition

    FROM matches m
    INNER JOIN pk_count pc
        ON UPPER(m.parent_database) = UPPER(pc.database_name)
       AND UPPER(m.parent_schema)   = UPPER(pc.schema_name)
       AND UPPER(m.parent_table)    = UPPER(pc.table_name)
    GROUP BY
        m.parent_database,
        m.parent_schema,
        m.parent_table,
        m.child_database,
        m.child_schema,
        m.child_table
)
SELECT
    parent_schema || '.' || parent_table AS "From Entity",
    child_schema || '.' || child_table   AS "To Entity",

    CASE
        WHEN matched_pk_columns = parent_total_pk_columns
            THEN 'Inferred 1-to-Many - Full PK Match'
        ELSE 'Possible Relationship - Partial PK Match'
    END AS "Relationship Type",

    join_condition AS "Join Condition",

    child_schema || '.' || child_table AS "Table Name",

    CASE
        WHEN matched_pk_columns = parent_total_pk_columns
            THEN 'Child entity contains all primary key columns from parent entity. Relationship inferred because FK is not defined in Snowflake.'
        ELSE 'Child entity contains only some primary key columns from parent entity. Needs business/data validation before finalizing.'
    END AS "Purpose",

    matched_pk_columns,
    parent_total_pk_columns

FROM relationship_group
ORDER BY
    "From Entity",
    "To Entity";
