# showcasehdi
HANA Showcase HDI

Exemplos de grant específicos para a tabela:
{   "role":{   "name": "Localizacao_SELECT_WITH_GRANT_OPTION#",   "object_privileges":[   {      "name":"TB.Localizacao",      "type": "TABLE",      "privileges_with_grant_option":["SELECT"]   }   ]   } }
{   "role":{   "name": "TransacaoFinanceira_SELECT",   "object_privileges":[   {      "name":"TransacaoFinanceiraCV",      "type": "VIEW",      "privileges":["SELECT"]   }   ]   } }

Exemplos de grants com diferentes estratégias:
Dar grants como ADMIN do HDI:

set schema "SHOWCASEHDI_1#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI_1#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI_1#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'showcasehdischema', '', 'SAPADMIN' );
--INSERT INTO SHOWCASEHDI_1#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'businessuser', '', 'SAPADMIN' );
--INSERT INTO SHOWCASEHDI_1#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'customerunmask', '', 'SAPADMIN' );
INSERT INTO SHOWCASEHDI_1#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'cardunmask', '', 'SAPADMIN' );
--CALL SHOWCASEHDI_1#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI_1#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI_1#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI_1#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI_1#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
create local temporary column table "#PRIVILEGES" like "_SYS_DI"."TT_SCHEMA_PRIVILEGES"; 
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('SELECT', '', 'SAPADMIN');
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('INSERT', '', 'SAPADMIN');
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('UPDATE', '', 'SAPADMIN');
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('DELETE', '', 'SAPADMIN');
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('REFERENCES', '', 'SAPADMIN');
insert into "#PRIVILEGES" ("PRIVILEGE_NAME", "PRINCIPAL_SCHEMA_NAME", "PRINCIPAL_NAME") values ('EXECUTE', '', 'SAPADMIN');
--call "SHOWCASEHDI#DI"."GRANT_CONTAINER_SCHEMA_PRIVILEGES"("#PRIVILEGES", "_SYS_DI"."T_NO_PARAMETERS", ?, ?, ?);
call "SHOWCASEHDI#DI"."REVOKE_CONTAINER_SCHEMA_PRIVILEGES"("#PRIVILEGES", "_SYS_DI"."T_NO_PARAMETERS", ?, ?, ?);
drop table "#PRIVILEGES";

-- Criação dos remote sources linkeddatabase optimized
CREATE REMOTE SOURCE "SAOLEOPOLDO" ADAPTER "hanaodbc" CONFIGURATION '<?xml version="1.0" encoding="UTF-8"?>
<ConnectionProperties name="connection_properties">
<PropertyEntry name="adapterversion">1.0</PropertyEntry>
<PropertyEntry name="connectionmode">Adapter Properties</PropertyEntry>
<PropertyEntry name="driver">libodbcHDB.so</PropertyEntry>
<PropertyEntry name="server">saoleopoldo</PropertyEntry>
<PropertyEntry name="port">30015</PropertyEntry>
<PropertyEntry name="dml_mode">readwrite</PropertyEntry>
<PropertyEntry name="linkeddatabase_mode">optimized</PropertyEntry>
<PropertyEntry name="extraadapterproperties"></PropertyEntry>
</ConnectionProperties>' WITH CREDENTIAL TYPE 'PASSWORD' USING 'user=SAPADMIN;password=xxx';

CREATE REMOTE SOURCE "PALOALTO" ADAPTER "hanaodbc" CONFIGURATION '<?xml version="1.0" encoding="UTF-8"?>
<ConnectionProperties name="connection_properties">
<PropertyEntry name="adapterversion">1.0</PropertyEntry>
<PropertyEntry name="connectionmode">Adapter Properties</PropertyEntry>
<PropertyEntry name="driver">libodbcHDB.so</PropertyEntry>
<PropertyEntry name="server">paloalto</PropertyEntry>
<PropertyEntry name="port">30015</PropertyEntry>
<PropertyEntry name="dml_mode">readwrite</PropertyEntry>
<PropertyEntry name="linkeddatabase_mode">optimized</PropertyEntry>
<PropertyEntry name="extraadapterproperties"></PropertyEntry>
</ConnectionProperties>' WITH CREDENTIAL TYPE 'PASSWORD' USING 'user=SAPADMIN;password=xxx';

call GET_REMOTE_SOURCE_PROPERTIES ('SQLSERVER', ?);

MER_POSITION			: hana.ST_POINT(4326); --or 3857

--CREATED ON HANA COCKPIT AS DBADMIN
--CREATE USER HDIGRANTOR PASSWORD <password> NO FORCE_FIRST_PASSWORD_CHANGE SET USERGROUP DEFAULT;
CREATE ROLE "sourcedata::external_access_g";
CREATE ROLE "sourcedata::external_access";
GRANT "sourcedata::external_access_g", "sourcedata::external_access" TO HDIGRANTOR WITH ADMIN OPTION;

--RUN AS SAPADMIN
GRANT SELECT, SELECT METADATA ON SCHEMA SAPADMIN               TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT ROLE ADMIN                                               TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SHOWCASEHC"       TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SYSRDL#CG_SOURCE" TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SQLSERVER"        TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "EXCEL"            TO "sourcedata::external_access_g" WITH GRANT OPTION;
GRANT SELECT, SELECT METADATA ON SCHEMA SAPADMIN               TO "sourcedata::external_access";


object owner "SHOWCASEHDI#OO" is not authorized to access the "PALOALTO.SAPADMIN.TESTE" synonym target; 
this user needs to be granted "SELECT" ("EXECUTE" for procedures) privileges on the target object as well as
 "LINKED DATABASE ON REMOTE SOURCE" privilege on remote source "PALOALTO". [8250528] 
Rodar como SAPADMIN
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SHOWCASEHC" TO "SHOWCASEHDI_1#OO";
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SYSRDL#CG_SOURCE" TO "SHOWCASEHDI_1#OO";
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "SQLSERVER" TO "SHOWCASEHDI_1#OO";
GRANT CREATE VIRTUAL TABLE ON REMOTE SOURCE "EXCEL" TO "SHOWCASEHDI_1#OO";
GRANT LINKED DATABASE ON REMOTE SOURCE "PALOALTO" TO SHOWCASEHDI_1#OO;
ALTER SYSTEM ALTER CONFIGURATION ('indexserver.ini', 'SYSTEM') SET ('smart_data_access', 'enable_xda_view_unfolding') = 'false' WITH RECONFIGURE;

Internal Presales DWC Agents
=============================
HANA_SAOLEOPOLDO_AGT
zeus.hana.prod.us-east-1.whitney.dbaas.ondemand.com
22497
DP_MSG_HANA_SAOLEOPOLDO_AGT
AxvPnS_);>{AuS8;

HANA_PALOALTO
zeus.hana.prod.us-east-1.whitney.dbaas.ondemand.com
22497
DP_MSG_HANA_PALOALTO
i9Q5c#X/rE]75Nd]

--NSE:https://blogs.sap.com/2019/07/22/increase-hana-data-capacity-with-sap-hana-native-storage-extensionnse./
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" 
PARTITION BY RANGE (year("CT_DATE"))
	((partition '2006' <= values < '2018', 
	  partition '2018' <= values < '2021',
	  partition others));			 
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" ALTER PARTITION 1 PAGE LOADABLE;
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" ALTER PARTITION 2 PAGE LOADABLE;
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" ALTER PARTITION 2 COLUMN LOADABLE;
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" MERGE PARTITIONS;
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" PAGE LOADABLE CASCADE;
ALTER TABLE "SHOWCASEHDI"."TB.CARD_TRANSACTIONS" COLUMN LOADABLE CASCADE;

grant AFL__SYS_AFL_APL_AREA_EXECUTE to SHOWCASEHDI#OO;
grant AFLPM_CREATOR_ERASER_EXECUTE to SHOWCASEHDI#OO;
call _SYS_REPO.GRANT_ACTIVATED_ROLE ('sap.pa.apl.base.roles::APL_EXECUTE', SHOWCASEHDI#OO);

			MER_POSITION2			: hana.ST_POINT(3857);
			MER_POSITION4			: hana.ST_GEOMETRY(3857);

https://saoleopoldo.demo.local:30030/
Pendencias:
Otimizar datatypes apos avaliar modelos estatísticos, principalmente o valor, flag fraude e estatus na tabela de transacoes
Otimizar datatypes apos avaliar modelos estatísticos, principalmente o valor, flag fraude e estatus na tabela de transacoes
falta masking n HDBtable
masking and NSE in HDI?
descobrir como debugar plano de execucao de calc views


Documentações:
https://jetcloud01proda21657417.hana.ondemand.com/public/sap/docs/hana/ide/help/default.html?a83fe9b8de1c4f4bbee3eea675851a04.html
https://help.sap.com/viewer/3823b0f33420468ba5f1cf7f59bd6bd9/2.0.04/en-US/625d7733c30b4666b4a522d7fa68a550.html
https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/2.0.04/en-US/1eb7eace0ee24021b1656e445322fed5.html
https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/2.0.04/en-US/df46a790f0694b0d9820487b385d138c.html
https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/2.0.04/en-US/ab883fca1e4047abafb7738b06131e46.html
https://help.sap.com/viewer/4505d0bdaf4948449b7f7379d24d0f0d/2.0.04/en-US/556452cac83f423597d3a38a6f225e4b.html
https://developers.sap.com/mission.xsa-get-started.html

*/


/*
SC_BUSINESSUSER - Ordinary business analyst, follows GDPR restrictions: masking, anonymization and hidden columns for customer data. Access only metadata but not financial transactions
SC_FINANCIALUSER - Ordinary financial analyst, follows some GDPR restrictions: masking, but no anonymization or hidden columns for customer data. Access metadata and financial transactions
SC_BUSINESSMANAGER - Manager for business, does not follow all GDPR restrictions: Access to unsmasked and non-anonymazed customer's data, but keeps masking for credit card numbers
SC_FINANCIALMANAGER - Manager for financials, does not follow all GDPR restrictions: Access to unsmasked credit card numbers, but keeps masking for customer's data
SC_ASSISTANTSEGMENT1 - Assistant, follows all GDPR restrictions + only customers on segment 1: Access to masked credit card numbers and customer's data.
SC_ASSISTANTCUSTOMER1 - Assistant, follows some GDPR restrictions and has access only for customer ID 1 (row level + column level restriction): Only access customer's data, with no masking.
SC_DBA - IT profile, has access all views still protected by GDPR
SC_SUPERUSER - IT superuser profile, has access all information, GDPR protected only by analytic privileges

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'businessuser', '', 'SC_BUSINESSUSER' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'financialuser', '', 'SC_FINANCIALUSER' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;


set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'businessuser', '', 'SC_BUSINESSMANAGER' );
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'customerunmask', '', 'SC_BUSINESSMANAGER' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'financialuser', '', 'SC_FINANCIALMANAGER' );
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'cardunmask', '', 'SC_FINANCIALMANAGER' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'assistantsegment1', '', 'SC_ASSISTANTSEGMENT1' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'assistantcustomer1', '', 'SC_ASSISTANTCUSTOMER1' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'dba', '', 'SC_DBA' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

set schema "SHOWCASEHDI#DI";
CREATE LOCAL TEMPORARY COLUMN TABLE SHOWCASEHDI#DI.#ROLES LIKE _SYS_DI.TT_SCHEMA_ROLES;
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'showcasehdischema', '', 'SC_SUPERUSER' );
INSERT INTO SHOWCASEHDI#DI.#ROLES ( ROLE_NAME, PRINCIPAL_SCHEMA_NAME, PRINCIPAL_NAME ) VALUES ( 'customerunmask', '', 'SC_SUPERUSER' );
--CALL SHOWCASEHDI#DI.REVOKE_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
CALL SHOWCASEHDI#DI.GRANT_CONTAINER_SCHEMA_ROLES( SHOWCASEHDI#DI.#ROLES, _SYS_DI.T_NO_PARAMETERS, ?, ?, ?);
DROP TABLE SHOWCASEHDI#DI.#ROLES;

Geo e Spatial demo: https://spatialdemos.cfapps.sap.hana.ondemand.com/




-- HDL HANA Data Lake
CALL SYSRDL#CG.REMOTE_EXECUTE('
BEGIN
    DROP TABLE CARD_TRANSACTIONS_COLD;
END');

CALL SYSRDL#CG.REMOTE_EXECUTE('
BEGIN
    CREATE TABLE CARD_TRANSACTIONS_COLD ( 
	"CT_ID" 		INTEGER NOT NULL,
	"CT_DATE"		DATE	NOT NULL,
	"CT_CARD_ID"	INTEGER NOT NULL,
	"CT_CUST_ID"	INTEGER NOT NULL,
	"CT_MER_ID" 	INTEGER NOT NULL,
	"CT_AMOUNT" 	DOUBLE	NOT NULL,
	"CT_FLAG_FRAUD" INTEGER,
	"CT_STATUS" 	INTEGER,
	PRIMARY KEY ("CT_ID")
    );
END');

CREATE VIEW HDL.VW_CARD_TRANSACTIONS AS 
SELECT * FROM "SHOWCASEHDI_1"."CARD_TRANSACTIONS_HOTWARM"
UNION ALL
SELECT * FROM "HDL"."CARD_TRANSACTIONS_COLD";


SELECT * FROM TABLE_PARTITIONS;
SELECT * FROM PARTITIONED_TABLES;
SELECT * FROM M_TABLE_PARTITIONS;
SELECT * FROM M_CS_TABLES WHERE SCHEMA_NAME = 'SHOWCASEHDI_1';

SELECT DISTINCT YEAR(CT_DATE) FROM "HDL"."VW_CARD_TRANSACTIONS";
SELECT DISTINCT YEAR(CT_DATE) FROM "HDL"."CARD_TRANSACTIONS_COLD";
SELECT DISTINCT YEAR(CT_DATE) FROM "SHOWCASEHDI_1"."CARD_TRANSACTIONS_HOTWARM";
SELECT DISTINCT YEAR(CT_DATE) FROM "SHOWCASEHDI_1"."CARD_TRANSACTIONS";

select count(*) from "SHOWCASEHDI_1"."CARD_TRANSACTIONS_HOTWARM";
select count(*) from "HDL"."CARD_TRANSACTIONS_COLD";
select count(*) from "HDL"."VW_CARD_TRANSACTIONS";


INSERT INTO "SHOWCASEHDI_1"."CARD_TRANSACTIONS" (SELECT * FROM "SAPADMIN"."CARD_TRANSACTIONS" ORDER BY 1);
SELECT SUM(CT_AMOUNT) FROM "HDL"."VW_CARD_TRANSACTIONS" WHERE CT_DATE >= '2018-01-01';



DROP TABLE "SAPADMIN"."VT_CARD_TRANSACTIONS_TAX_20000";
DROP TABLE "SAPADMIN"."VT_CARD_TRANSACTIONS_TAX_10";

CREATE VIRTUAL TABLE "SAPADMIN"."VT_CARD_TRANSACTIONS_TAX_20000" AT "MSEXCEL"."<NULL>"."<NULL>"."card_transactions_tax.xlsx/Tax_20000";
CREATE VIRTUAL TABLE "SAPADMIN"."VT_CARD_TRANSACTIONS_TAX_10"    AT "MSEXCEL"."<NULL>"."<NULL>"."card_transactions_tax.xlsx/Tax_10";



SELECT 
USER_NAME,
LAST_EXECUTION_TIMESTAMP,
STATEMENT_STRING,
APPLICATION_SOURCE,
APPLICATION_NAME
FROM M_SQL_PLAN_CACHE WHERE STATEMENT_STRING LIKE '%CUST_STATE%'
ORDER BY LAST_EXECUTION_TIMESTAMP DESC;

./hdbinst --silent --batch --path="/usr/sap/dataprovagent0" --agent_listener_port=30090 --agent_admin_port=30091
./agentcli.sh --configAgent
./agentcli.sh --setSecureProperty

http://hdbgraph.wdf.sap.corp:8000/sap/hana/spatial/demos/demo_europe/index.html

ALTER VIRTUAL TABLE XXX REFRESH DEFINITION;
package.json para refresh nas estruturas das remote virtual tables
{
    "name": "deploy",
    "dependencies": {
        "@sap/hdi-deploy": "^3.*"
    },
    "scripts": {
        "start": "node node_modules/@sap/hdi-deploy/deploy.js --treat-unmodified-as-modified"
    }
}
 --parameter validate_virtual_tables=true
 
The customer can implement the masking themselves. The best approach would be to create a SQL function with the masking type as a parameter for numeric columns directly in the view definition. 
In the function, the customer is free to define what is returned according to the authorization of the session user (or current user for definer mode). 
The authorization can be managed simply and with good performance in a mapping table (UserName, Mask-Value). Our development recommends to only enforce top-level views for session users. 