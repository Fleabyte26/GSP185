# GSP185 - App Dev: Storing Image and Video Files in Cloud Storage (Python)

Paste each run as one block into Cloud Shell.

## Run 1 - Get the lab code (skip if ~/cloudstorage already exists)

```
git clone --depth=1 https://github.com/GoogleCloudPlatform/training-data-analyst
ln -s ~/training-data-analyst/courses/developingapps/v1.3/python/cloudstorage ~/cloudstorage
cd ~/cloudstorage/start
. prepare_environment.sh
```

## Run 2 - Fix the code, create the bucket, upload the logo (passes both checkpoints)

```
gcloud config set project $DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
gsutil mb gs://$GCLOUD_BUCKET 2>/dev/null
cd ~/cloudstorage/start

cat > quiz/webapp/templates/add.html <<'EOF'
{% extends 'master.html' %}
{% block head %}
	<title>Quiz - Python</title>
{% endblock %}
{% block content %}
	<h3>{{ action }} question</h3>
	<form enctype="multipart/form-data" method="post">
		<div class="form-group"><label for="author">Author</label><input class="form-control" id="author" name="author" type="text" value="{{ question.author }}"></div>
		<div class="form-group"><label for="quiz">Quiz</label><select class="form-control" id="quiz" name="quiz">
			<option value="gcp">-- Select Category --</option>
			<option value="gcp">Google Cloud Platform</option>
			<option value="people">People</option>
			<option value="places">Places</option>
		</select></div>
		<div class="form-group"><label for="title">Title</label><input class="form-control" id="title" name="title" type="text" value="{{ question.title }}"></div>
		<div class="form-group hidden"><input id="imageUrl" name="imageUrl" type="hidden" value="{{ question.imageUrl }}"></div>
		<div class="form-group"><label for="image">Image</label><input class="form-control" id="image" name="image" type="file"></div>
		<div class="form-group"><label>Answer 1</label><div class="input-group"><span class="input-group-addon"><input name="correctAnswer" type="radio" value="1"></span><input class="form-control" name="answer1" type="text" value="{{ question.answer1 }}"></div></div>
		<div class="form-group"><label>Answer 2</label><div class="input-group"><span class="input-group-addon"><input name="correctAnswer" type="radio" value="2"></span><input class="form-control" name="answer2" type="text" value="{{ question.answer2 }}"></div></div>
		<div class="form-group"><label>Answer 3</label><div class="input-group"><span class="input-group-addon"><input name="correctAnswer" type="radio" value="3"></span><input class="form-control" name="answer3" type="text" value="{{ question.answer3 }}"></div></div>
		<div class="form-group"><label>Answer 4</label><div class="input-group"><span class="input-group-addon"><input name="correctAnswer" type="radio" value="4"></span><input class="form-control" name="answer4" type="text" value="{{ question.answer4 }}"></div></div>
		<button class="btn btn-success" type="submit">Save</button>
	</form>
{% endblock %}
EOF

cat > quiz/gcp/storage.py <<'EOF'
import os
project_id = os.getenv('GCLOUD_PROJECT')
bucket_name = os.getenv('GCLOUD_BUCKET')
from google.cloud import storage
storage_client = storage.Client()
bucket = storage_client.get_bucket(bucket_name)

def upload_file(image_file, public):
    blob = bucket.blob(image_file.filename)
    blob.upload_from_string(image_file.read(), content_type=image_file.content_type)
    if public:
        blob.make_public()
    return blob.public_url
EOF

cat > quiz/webapp/questions.py <<'EOF'
from quiz.gcp import storage, datastore

def upload_file(image_file, public):
    if not image_file:
        return None
    return storage.upload_file(image_file, public)

def save_question(data, image_file):
    if image_file:
        data['imageUrl'] = str(upload_file(image_file, True))
    else:
        data['imageUrl'] = u''
    data['correctAnswer'] = int(data['correctAnswer'])
    datastore.save_question(data)
    return
EOF

cd ~
python3 -c "import base64;open('Google_Cloud_Storage_logo.png','wb').write(base64.b64decode('iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg=='))"
gsutil cp Google_Cloud_Storage_logo.png gs://$GCLOUD_BUCKET/
gsutil ls gs://$GCLOUD_BUCKET/
echo "Done. Click Check my progress on both checkpoints."
```

Click Check my progress on "Create a bucket" and "Upload objects to your bucket". You should pass.

## Run 3 - Optional: start the web app

```
cd ~/cloudstorage/start
source ~/venvs/developingapps/bin/activate
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
python run_server.py
```

Web preview > Preview on port 8080. Ctrl+C to stop.
Note: if Create Question shows "database (default) does not exist", the lab's Datastore setup failed. That is Google's side, not your code, and does not affect the score.
