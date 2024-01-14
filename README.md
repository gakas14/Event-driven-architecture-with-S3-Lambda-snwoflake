# Event-driven-architecture-with-S3-Lambda-snowflake
In this project, a lambda function will be triggered when a CSV file is uploaded into a bucket(source bucket); the function will extract the CSV file and load the data in a pandas data frame. Afterward, it will remove some unnecessary characters and then save the data in another S3 bucket(destination bucket), triggering the Snowpipe to load the newly created file automatically into a table in Snowflake.

![pjt_eda drawio](https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/fb817e33-ad3f-461f-bab2-74c95669529d)




## Step 1: Create an s3 bucket as the source bucket to upload the CSV file and another bucket as a destination to save the transform file. 

### Source bucket
<img width="1257" alt="Screen Shot 2024-01-11 at 11 42 40 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/dabd3eba-3724-4ba6-b258-bc67d2e8e900">

### Destination bucket
<img width="1157" alt="Screen Shot 2024-01-11 at 11 43 31 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/d7e8c621-ba26-44be-818d-1aa9acf55742">


## Step 2: Create a lambda role and a lambda function. And add two lambda layers to execute the different libraries. 

### Create a lambda role 
<img width="756" alt="Screen Shot 2024-01-11 at 11 46 30 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/7d79ac5d-7ac0-4dc2-b552-024ef70eca7c">

#### with s3 and cloud watch access

<img width="768" alt="Screen Shot 2024-01-11 at 11 48 14 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/3ca17d7d-99fd-45bc-a51f-3f690c88034f">

### Create the lambda function 
<img width="1131" alt="Screen Shot 2024-01-11 at 11 50 14 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/f58054c8-0613-4251-ad06-c12e6efbc0cf">

#### Add the source bucket as a trigger
<img width="1074" alt="Screen Shot 2024-01-11 at 11 51 24 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/c82dfd48-2561-4a6b-bbe6-ae0b7cc40f70">

#### Add two lambda layers
<img width="1075" alt="Screen Shot 2024-01-11 at 11 52 18 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/5fc9e66d-1e4d-480c-b89e-1accddff84fc">

#### Python script for the lambda 
<sub> 
  
      import pandas as pd
      from io import StringIO
      import boto3
      # import os
      # import csv
      # from csv import reader
      # import io
      # from io import BytesIO
      # import traceback
      s3 = boto3.client('s3')
      def lambda_handler(event, context):
          #print(event)
          # Get the bucket and the file name
          bucket = event['Records'][0]['s3']['bucket']['name']
          key = event['Records'][0]['s3']['object']['key']
      
          print(bucket)
          print(key)
      
          # Get the object
          response = s3.get_object(Bucket=bucket, Key=key) 
      
          # Process the data
          data = response['Body'].read().decode('utf-8')
          df = StringIO(data)
          initial_df= pd.read_csv(df, sep=",")
          #print(df['Average_Covered_Charges'])
      
      
          # Remove ',' in the column street address 
          initial_df['Street_Address_'] = initial_df['Street_Address_'].str.replace(',', ' ')
          print(initial_df['Street_Address_'])
        
      
          dest_bucket =  'gakas-snowflake-zero-to-hero-masterclass'
          dest_key = 'snowflake/csv/health_transform.csv'
          csv_buffer = StringIO()
          initial_df.to_csv(csv_buffer, index=False)
      
          # Save the transform file 
          s3.put_object(Body=csv_buffer.getvalue(), Bucket=dest_bucket , Key=dest_key) 
</sub>



## Step 3: Upload the CSV file  in the source bucket and check the destination bucket to get the transform file. 

### Source bucket
<img width="1177" alt="Screen Shot 2024-01-11 at 11 57 24 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/67886813-e06e-432a-bf9c-deff7092189f">

#### destination bucket
<img width="1243" alt="Screen Shot 2024-01-11 at 11 58 07 AM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/c2914189-c787-4a74-a215-bfb81cee203a">


## Step 4: Configure the snowflake to connect with the destination bucket. 

<sub>


      --drop database pjt_eda;
      --Database Creation 
      create database if not exists pjt_eda;
      use pjt_eda;

      --Table Creation
      CREATE or replace TABLE  PJT_EDA.PUBLIC.HEALTHCARE (
            AVERAGE_COVERED_CHARGES    NUMBER(38,6)  
           ,AVERAGE_TOTAL_PAYMENTS    NUMBER(38,6)  
           ,TOTAL_DISCHARGES    NUMBER(38,0)  
           ,BACHELORORHIGHER    NUMBER(38,1)  
           ,HSGRADORHIGHER    NUMBER(38,1)  
           ,TOTALPAYMENTS    VARCHAR(128)  
           ,REIMBURSEMENT    VARCHAR(128)  
           ,TOTAL_COVERED_CHARGES    VARCHAR(128) 
           ,REFERRALREGION_PROVIDER_NAME    VARCHAR(256)  
           ,REIMBURSEMENTPERCENTAGE    NUMBER(38,9)  
           ,DRG_DEFINITION    VARCHAR(256)  
           ,REFERRAL_REGION    VARCHAR(26)  
           ,INCOME_PER_CAPITA    NUMBER(38,0)  
           ,MEDIAN_EARNINGSBACHELORS    NUMBER(38,0)  
           ,MEDIAN_EARNINGS_GRADUATE    NUMBER(38,0)  
           ,MEDIAN_EARNINGS_HS_GRAD    NUMBER(38,0)  
           ,MEDIAN_EARNINGSLESS_THAN_HS    NUMBER(38,0)  
           ,MEDIAN_FAMILY_INCOME    NUMBER(38,0)  
           ,NUMBER_OF_RECORDS    NUMBER(38,0)  
           ,POP_25_OVER    NUMBER(38,0)  
           ,PROVIDER_CITY    VARCHAR(128)  
           ,PROVIDER_ID    NUMBER(38,0)  
           ,PROVIDER_NAME    VARCHAR(256)  
           ,PROVIDER_STATE    VARCHAR(128)  
           ,PROVIDER_STREET_ADDRESS    VARCHAR(256)  
           ,PROVIDER_ZIP_CODE    NUMBER(38,0)  
        );

       --Create an integration object for the external stage
       create or replace storage integration pjt_EDA_integration
        type = external_stage
        storage_provider = s3
        enabled = true
        storage_aws_role_arn = 'arn:aws:iam::200105849428:role/Snowflake_role'
        storage_allowed_locations = ('s3://gakas-snowflake-zero-to-hero-masterclass/snowflake/');

  
      --Describe integration object to fetch external_id and to be used in s3
      DESC INTEGRATION pjt_EDA_integration;

      -- Create a file object
      Create or replace file format PJT_EDA.public.csv_format
                    type = csv
                    --field_delimiter = '|'
                    skip_header = 1
                    null_if = ('NULL', 'null')
                    empty_field_as_null = true;


      -- Create a stage object
      Create or replace stage PJT_EDA.public.ext_csv_stage
        URL = 's3://gakas-snowflake-zero-to-hero-masterclass/snowflake/csv'
        STORAGE_INTEGRATION = pjt_EDA_integration
        file_format = PJT_EDA.public.csv_format;
 
      list @ext_csv_stage;
 
      -- Create a pipe
      create or replace pipe PJT_EDA.PUBLIC.pjt_eda_pipe 
       auto_ingest=true as copy into PJT_EDA.PUBLIC.HEALTHCARE from 
       @PJT_EDA.PUBLIC.ext_csv_stage FILE_FORMAT=(FORMAT_NAME=csv_format);
 
      show pipes;
 
      select * from PJT_EDA.PUBLIC.HEALTHCARE;
      select count(*) from PJT_EDA.PUBLIC.HEALTHCARE;
 
 
      delete from PJT_EDA.PUBLIC.HEALTHCARE;
</sub>

### Add an event on the Destination bucket to trigger Snwopipe

<img width="1229" alt="Screen Shot 2024-01-14 at 2 24 25 PM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/4085b4f5-6c4c-4e61-a4e9-e58b3aec830d">
 
### Show the table in snowflake

<img width="1272" alt="Screen Shot 2024-01-14 at 3 53 33 PM" src="https://github.com/gakas14/Event-driven-architecture-with-S3-Lambda-snwoflake/assets/74584964/be8f0943-854d-4889-b4f6-c84f42d0a487">



