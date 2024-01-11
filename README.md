# Event-driven-architecture-with-S3-Lambda-snowflake
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
