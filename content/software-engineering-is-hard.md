+++
title = 'Software Engineering Is Hard'
date = 2025-11-22T16:51:31-05:00
draft = true
+++

software engineering is hard. For instance, I added a dry run flag, and now my logs are invalid (excerpt):

```diff
 CACHE_DIR = os.environ["CACHE_DIR"]
 DATABASE_URI = f"sqlite:///{DB_PATH}"
+DRY_RUN = os.environ.get("DRY_RUN", "false").lower() == "true"
 
 
 s3 = boto3.client("s3")
 
 
 # app entry point for aws lambda
 def handler(event, context):
     """AWS Lambda entrypoint."""
     try:
         # Make sure /tmp DB is ready
@@ -86,54 +86,55 @@ def handler(event, context):
             )
             """
         )
         conn.commit()
 
         update_file_list(conn)  # job
         fetch_comp_reports(conn)  # job
         fetch_manual_orders(conn)  # job
         cast_tables(conn)  # job
         build_views(conn)  # job
-        create_google_sheet_from_sqlite_views(conn)  # job
+        if not DRY_RUN:
+            create_google_sheet_from_sqlite_views(conn)  # job
 
         conn.close()
 
         # Save back to S3
-        persist_db()
+        if not DRY_RUN:
+            persist_db()
 
         return {"statusCode": 200, "body": "DB updated"}
 
     except Exception as e:
         # Explicit 500 on errors
         return {"statusCode": 500, "body": f"Internal error: {e}"}

```

my brain says "monad!", or at least "errors as values!".

But these kinds of "memes" are just stand-ins for poorly drawn maps of software engineering concepts. I wish the matrix was a thing. Not an all knowing AI that can tell me what I'm doing wrong, but an "upload into the brain", of millenia of simulated software engineering experience. To just say, "I know kung fu."

Alas, we don't live in that world, so I'm forced to deconstruct the concept behind these memes. And I'm liable to forget them again, if I don't practice them enough, too. I'll get lucky by picking the right pattern some of the time, but Chesterton's fence will get me when I switch back to the old pattern. It's why I want to start blogging some of my learnings. But it always feels like a waste when I currently "know the thing". It feels like you'll know it forever. But you won't. Document your damn code. And write a blog post about it, too.

As for this -- I probably want to error, value tuples or something equivalent-but-more-pythonic, and then use it to mutate some status object, that I return at the end of the pipeline. But I don't know what the best pattern is. Software engineering is hard.


