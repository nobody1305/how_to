1. make sure that your deleted bucket still on soft deleted period.
2. go to gcp cli and run this command

'''
gcloud storage ls --buckets --soft-deleted --full > output.txt
'''
   and you can download the result from cloud shell
4. after get the result collect bucket name and generation number. you can restore your bucket using this command
'''
gcloud storage restore gs://BUCKET_NAME#GENERATION_NUMBER
'''
5. you can restore all object inside it using
'''
gcloud storage restore gs://BUCKET_NAME/FOLDER/OBJECT
'''
   or
'''
gcloud storage restore gs://BUCKET_NAME/**
'''
   for all object inside bucket
