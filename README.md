<h1>Serverless Text To Speech Web application</h1>

 ### [Text To Speech Web application](https://daa81ie6rht9c.cloudfront.net/)

<h2>Overview</h2>
A Text-to-Speech (TTS) converter that transforms text into natural, lifelike speech. This solution demonstrates how cloud technologies can transform textual data into lifelike speech, enhancing accessibility and user engagement.

<h2>Architecture Highlights</h2>

- **Amazon S3**: Hosts the static website and stores generated audio files for easy retrieval.
- **Amazon CloudFront**: Ensures fast and secure content delivery for the website and audio files.
- **Amazon API Gateway**: Provides a secure interface for user requests to interact with backend services.
- **AWS Lambda**: Executes the backend logic for handling TTS requests and invoking services.
- **Amazon Polly**: Converts text into natural-sounding speech.
<br/>
<p align="center">
  <img src="https://i.imgur.com/obDkHXc.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br/>

<h2>Environments Used </h2>

- <b>AWS Console</b>
- <b>Visual Studio Code</b>

<h2>Creating an S3 Bucket:</h2>
<ol>
  <li>In the S3 homepage, click <strong>“Create bucket”</strong>.</li>
  <li>Add a unique bucket name, e.g., <strong>texttospeechbucket</strong>.</li>
  <li>Leave the default settings and click <strong>“Create bucket”</strong>.</li>
  <li>In the <strong>“Properties”</strong> tab of the created bucket, scroll down and enable <strong>Static Website Hosting</strong>.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/7xuj7kP.png" height="80%" width="80%" alt="S3 Steps"/>
</p>
<br/>

<h2>Creating the Lambda function:</h2>
<ol>
  <li>In the Amazon Lambda homepage, click <strong>“Create function”</strong>.</li>
  <li>Add a function name and select Python for the runtime.</li>
  <li>Leave all other settings as default and click <strong>“Create function”</strong>.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/WuRyA27.png" height="80%" width="80%" alt="Lambda Steps"/>
</p>
<br/> 

<ol start="4">
  <li>In the Lambda homepage, go to functions and click on your function.</li>
  <li>In the “Code” tab, copy the following code into the Python file. This code will use Amazon Polly to convert text to speech and store it as an MP3 file in a folder in the S3 bucket. It also includes error handling for debugging.</li>
</ol>



### Python Code: Lambda Function for Text-to-Speech

```python
import json
import boto3
import os
import time

def lambda_handler(event, context):
    # Initialize AWS Polly and S3 clients
    polly_client = boto3.client('polly')
    s3_client = boto3.client('s3')
    
    # Get input parameters from the event payloads
    text = event.get('text', None)  # Text to convert to speech

    bucket_name = '<YOUR-S3-BUCKET-NAME>'  # Replace with your S3 bucket name
    region = '<YOUR-REGION>'  # Replace with your bucket's region (e.g., 'ap-southeast-1')
    polly_voice = 'Joanna'  # Replace with your desired Polly voice (optional)
    polly_engine = 'neural'  # Use 'neural' or 'standard'

    # Validate input text
    if not text:
        return {
            'statusCode': 400,
            'headers': {
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'POST',
                'Access-Control-Allow-Headers': 'Content-Type'
            },
            'body': json.dumps({'error': 'No text provided'})
        }

    # Synthesize speech using Polly
    try:
        response = polly_client.synthesize_speech(
            Text=text,
            OutputFormat='mp3',
            VoiceId=polly_voice,
            Engine=polly_engine
        )
    except Exception as e:
        return {
            'statusCode': 400,
            'headers': {
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'POST',
                'Access-Control-Allow-Headers': 'Content-Type'
            },
            'body': json.dumps({'error': str(e)})
        }

    # Create a unique filename for the audio file
    timestamp = int(time.time() * 1000)  # Current time in milliseconds
    file_name = f"speech_{timestamp}.mp3"
    file_path = f"/tmp/{file_name}"

    # Save the audio stream to /tmp in Lambda
    if "AudioStream" in response:
        with open(file_path, 'wb') as file:
            file.write(response['AudioStream'].read())

    # Define the S3 key (path) for the file
    s3_key = f'speech/{file_name}'

    # Upload the file to S3
    try:
        s3_client.upload_file(file_path, bucket_name, s3_key)
        print(f"Uploaded file to S3 with key: {s3_key}")
    except Exception as e:
        return {
            'statusCode': 400,
            'headers': {
                'Access-Control-Allow-Origin': '*',
                'Access-Control-Allow-Methods': 'POST',
                'Access-Control-Allow-Headers': 'Content-Type'
            },
            'body': json.dumps({'error': str(e)})
        }

    # Generate a public S3 URL for the uploaded file
    s3_url = f'https://{bucket_name}.s3.{region}.amazonaws.com/{s3_key}'

    # Return the S3 URL in the response
    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'POST',
            'Access-Control-Allow-Headers': 'Content-Type'
        },
        'body': json.dumps({
            'message': 'Text-to-speech conversion successful!',
            's3_url': s3_url
        })
    }

```

<h2>Creating an API Gateway:</h2>
<ol>
  <li>In the Amazon API Gateway homepage, click <strong>“Create API”</strong>.</li>
  <li>Select REST API.</li>
  <li>Add an API name and description + select <strong>“edge-optimized”</strong> for API endpoint type.</li>
  <li>Click <strong>“Create API”</strong>.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/BtZejrD.png" height="80%" width="80%" alt="API"/>
</p>
<br/> 

<ol start="5">
  <li>In the “resources” tab of your API, select <strong>“Create method”</strong>.</li>
  <li>Select the <strong>“GET”</strong> method, integration type as <strong>“Lambda function”</strong> and select your Lambda function.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/3MsvAC0.png" height="80%" width="80%" alt="methods"/>
</p>
<br/> 

<ol start="7">
  <li>Repeat this for an “OPTIONS” method and a “POST” method.</li>
  <li>In the resources tab of your API, click on <strong>“Enable CORS”</strong> (Cross Origin Resource Sharing).</li>
  <li>Check the GET, OPTIONS and POST checkboxes and click <strong>“Save”</strong>.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/KrOvqmu.png" height="80%" width="80%" alt="CORS"/>
</p>
<br/> 

<h2>Create a HTML file for the website with API integration:</h2>
<ol>
  <li>Create a new html document in an IDE such as VSCode and name it <strong>“index.html”</strong>.</li>
  <li>Code some simple HTML for the website with a text-box to enter text and a button to invoke a JavaScript function.</li>
  <li>Add the JavaScript to submit user input to the API Gateway. Make sure to replace the apiURL with your invoke URL, which can be found in the “Stages” tab of your API.</li>
</ol>

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text-to-Speech Converter</title>
</head>
<body>
    <h1>Text-to-Speech Converter</h1>
    <textarea id="text-input" placeholder="Enter text here..."></textarea>
    <button onclick="convertToSpeech()">Convert to Speech</button>
    <div id="result"></div>
    <audio id="audio-player" controls style="display:none;">
        <source id="audio-source" src="" type="audio/mpeg">
    </audio>

    <script>
        async function convertToSpeech() {
            const text = document.getElementById('text-input').value;
            if (!text) {
                alert('Please enter some text!');
                return;
            }

            const apiUrl = 'YOUR_API_GATEWAY_URL'; // Replace with your API Gateway endpoint

            const requestBody = { text: text };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(requestBody)
                });

                if (!response.ok) {
                    throw new Error('Error during conversion');
                }

                const result = await response.json();
                const responseBody = JSON.parse(result.body);

                if (!responseBody.s3_url) {
                    throw new Error('Received undefined URL');
                }

                document.getElementById('result').innerHTML = `
                    Conversion successful! You can download the MP3 file here:
                    <a href="${responseBody.s3_url}" target="_blank">Download Speech MP3</a>
                `;

                const audioPlayer = document.getElementById('audio-player');
                const audioSource = document.getElementById('audio-source');
                audioSource.src = responseBody.s3_url;
                audioPlayer.style.display = 'block';
                audioPlayer.load();
            } catch (error) {
                document.getElementById('result').innerHTML = `Error: ${error.message}`;
            }
        }
    </script>
</body>
</html>
```
<ol start="7">
  <li>In your S3 bucket, click on “Upload”.  Click “Add files” and select your “index.html” file.</li>
  <li>Finally click “Upload.</li>
</ol>

<h2> Creating a CloudFront distribution to host the static website</h2>
<ol>
  <li>In the CloudFront homepage, click on <strong>“Create distribution”</strong>.</li>
  <li>Select the S3 bucket as the Origin Domain.</li>
  <li>Check the second option for Origin access and click on “Create new OAC”, then “Create”.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/Kao7Ojb.png" height="80%" width="80%" alt="OAC"/>
</p>
<br/> 

<ol start="4">
  <li>Scroll down to WAF settings and select “Do not enable security protections”.</li>
  <li>Then click on “Create distribution” at the very bottom.</li>
  <li>AWS will provide you with a policy to copy. Copy this policy.</li>
</ol>
<br/>
<p align="center">
  <img src="https://i.imgur.com/k5fPCTn.png" height="80%" width="80%" alt="policy"/>
</p>
<br/> 
<ol start="7"> 
  <li>In the permissions tab of your S3 bucket, make sure that “Block public access” settings are off.</li>
  <li>In the bucket policy section, click on “Edit” and paste the policy.</li>
  <li>We will also need to implement the following policy to allow the website to retrieve audio files:</li>
</ol>
  <pre>
  {
    "Sid": "AllowPublicAccessToAudioFiles",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::S3_BUCKET_NAME/*"
  }
  </pre>
<ol start="10"> 
  <li>Save changes to the policy.</li>
  <li>Heading back into CloudFront, click on the distribution for your S3 bucket. Here you will see the distribution domain name which will now be used to access the static website.</li>
</ol>

<br/>
<p align="center">
  <img src="https://i.imgur.com/oQaFJ8O.png" height="60%" width="60%" alt="domain"/>
</p>
<br/>

<h2 align="center">
With some styling, the website is complete!:  <br/>
  <img src="https://i.imgur.com/207iBMX.png" height="80%" width="80%" alt="final"/>
</h2>
<br/>

