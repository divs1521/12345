Hi Jalpesh, I tried both Cortex function options in Snowflake:

1. 'AI_COMPLETE'
2. 'SNOWFLAKE.CORTEX.COMPLETE'

Both are failing with function-not-found errors in the current worksheet/session. This looks like Cortex is either not enabled for this account/region or my current role does not have access to Cortex AI functions.
Could you please confirm whether my role has Cortex access? I may need one of these grants:
'GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE <my_role>;'
and/or
'GRANT USE AI FUNCTIONS ON ACCOUNT TO ROLE <my_role>;'
Once access is available, I can pass the Customer and Interaction metadata/profiling output into Cortex to identify candidate primary keys and generate inferred ER relationships separately for both schemas.
