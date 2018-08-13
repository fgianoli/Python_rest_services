LOCAL CGI Script for Rest Services: Postgres to JSON

developed on cloud9

    Basic Ubuntu 14.04.5 LTS Workspace (Blank)
    PostgreSQL 9.3.18
    Python 2.7.6

Please check that .py files are executable ( step 00 )
If you install this form a git clone to a fresh cloud9 vm, do also step 0
otherwise jsut check to have already python 2.7x + psycopg2 module installed.

To clone to Cloud9 workspace directly see https://docs.c9.io/docs/setting-up-github-workspace

------------------
How To Setup and Run
------------------
00 - set file permnissions

    sudo chmod a+x rest.py
    sudo chmod a+x server.py
    sudo chmod a+x rest_doc.py

0 - install psycopg2

    sudo apt-get update
    sudo apt-get install libpq-dev python-dev
    sudo pip install psycopg2
    

    
1 - Import Protected areas shp using QGIS
    
    ADD POSTGIS TABLE(S) --> NEW
    Provide connection information --> OK
    DATABASE --> DBMANAGER
    IMPORT
    
2 - Create a function

    OPEN PGADMIN  
```
CREATE OR REPLACE FUNCTION f_wdpa_area(wdpaid bigint)
 RETURNS TABLE(wdpaid bigint, name text, gis_area numeric) AS 
 $$ 
 SELECT wdpaid,name,gis_area from public.wdpa WHERE wdpaid=$1 
$$ LANGUAGE SQL;
```
    
3 - Set DB connection in rest.py

    07#  DBNAME = 'postgres'
    08#  HOST = 'localhost'
    09#  USER = 'user'
    10#  PASSWORD = 'user'
    
5 - Go to the downloaded folder and start the CGI Script for Local Rest Service Server (set PORT in server.py)

    python server.py
    
6 - Test the JSON response:
    
    eg call a function that list the protected area with id = 916
   
        
        curl "http://localhost:8888/rest.py?type=fun&schema=public&obj=f_wdpa&params=(wdpaid:916)"
    
        that will print
        
        [
          {
            "wdpaid": "0", 
            "name": "Serengeti National Park"
            "gis_area": "13123.05"
          }
        ]
    
url - parameters:
    
    - type = tab (or) fun  //tab to call tables or views, fun to call functions
    
    - schema = schema_name // eg.public default postgres schema
    
    - obj = table_name (or) view_name (or) function_name
    
    - params = (param1:'val1', param2:'val2', ... paramN:'valN')  //Optional definition of function parameters
      note: parameters values are not quoted so you have to use single quote in the call right now; will add the quoting
            directly in the function calling postgres functions.
