# ai-blog-django-app

## Table of Contents

1. [About](#about)
2. [Notes](#notes)
3. [Tools](#tools)
4. [Screenshots](#Screenshots)
5. [Links](#links)

## About

An AI Blog web app made using Python and Django based off of a freecodecamp video [linked here](https://www.youtube.com/watch?v=ftKiHCDVwfA).

Normally I wouldn't have a repo with a project stemming from a video, but I had a few changes and fixes I wanted to make known in-case anyone else was having frustrations while following along the original tutorial, which will be addressed in the [notes](#notes) section.

This web app recieves a youtube link and using AI will generate a blog posts base on the video by doing the follow:

1. User enters the youtube link, 
2. Using [pytube](https://pytube.io/en/latest/), the web app downloads the video as an MP4, and converts it to a MP3 file internally. 
3. Once an MP3 file is ready, it uses [Assembly AI](https://www.assemblyai.com/) to extract the transripts of the audio as text.
4. That text is then fed into [Open AI](https://openai.com/) as part of a prompt
5. That response is then finally stored in the [SQLite 3](https://www.sqlite.org/index.html) database.

## Notes

Some issues and small changes I had:

### Issue #1:
pytube's streams was returning a 400 with an error message (urllib.error.HTTPError: HTTP Error 400: Bad Request), there was an open PR on pytube's repo with the fix, I had made changes to my local install according to [this](https://github.com/JuanBindez/pytubefix/commit/c0c07b046d8b59574552404931f6ce3c6590137d).

### Issue #2:
When using an env file, for some reason with Assembly AI's API key, I had to remove the '"' characters.

```python
aai.settings.api_key = os.getenv('AAI_AI_API_KEY').replace('"', "")
```

### Issue #3:
Open AI's free trial doesn't seem to be a thing anymore as I was getting a 429 with error message "openai.RateLimitError: Error code: 429". At some point I think I will look at an alternative free trial, but for now I simply mocked the response in a try except block. 

```python
try:
    prompt = f"Based on the following transcript from a YouTube video, write a comprehensive blog article, write it based on the transcript, but dont make it look like a youtube video, make it look like a proper blog article:\n\n{transcription}\n\nArticle:"

    client = OpenAI(api_key=os.getenv('OPEN_AI_API_KEY'))


    completion = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": prompt}
        ]
    )

    generated_content = completion.choices[0].message
except Exception as e:
    generated_content = f"This is a fake response as the chat gpt request failed due to {e}"
```

### Change #1:
While testing, I was getting annoyed with an error where using the same youtube link would throw an error since the video already existed, so I added a check using Django ORM.

```python
# check existing blog posts
if BlogPost.objects.filter(youtube_link=yt_link).exists():
    blog_article = BlogPost.objects.filter(youtube_link=yt_link).first()

    return JsonResponse({'content': blog_article.generated_content})
```

## Tools

1. [Python 3.12.4](https://www.python.org/downloads)
2. [Django 5.0.7](https://www.djangoproject.com/download)
3. [SQLite 3](https://www.sqlite.org/index.html)
4. [pytube](https://pytube.io/en/latest/)
5. [Assembly AI](https://www.assemblyai.com/)
6. [Open AI](https://openai.com/)


## Screenshots

### signup
[<img src="/screenshots/signup.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/signup.png)
### login
[<img src="/screenshots/login.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/login.png)
### admin
[<img src="/screenshots/admin.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/admin.png)
### index
[<img src="/screenshots/index.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/index.png)
### blog-list
[<img src="/screenshots/blog-list.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/blog-list.png)
### blog-post
[<img src="/screenshots/blog-post.png" align="left" width="369" hspace="5" vspace="10">](/screenshots/blog-post.png)
