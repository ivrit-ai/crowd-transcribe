# crowd-transcribe
Crowd transcription server


# Local Environment Setup

To work on this repo locally, please follow these steps.

## Install Dependencies

**Note:** If you're using a virtual environment, make sure it's activated.

```
pip install -r requirements.txt
```

## Acquire DB connection string

You will need to have access to PostgreSQL - Easiest is to install it locally.
See more [here](https://www.postgresql.org/download/).

Create an empty database for the project. Connection string should point to this database.

## Flask App Secret Generation

You can generate a local app secret for cookie signing using a simple script like this:

`python -c 'import secrets; print(secrets.token_hex())'`

See [here](https://flask.palletsprojects.com/en/3.0.x/quickstart/#sessions) for more info.

## Transcripts Directory

Starting the server, will require specifying a `--transcripts-dir` argument.
This argument should point to a directory containing JSON files with transcripts.

Each JSON file represents a repository of audio file transcripts, this can be thought of, for example, as an episode of some podcast, broken up to small audio snippets, each one with potentially some initial transcription for the user to review and improve on.

A basic JSON file might look something like this:

```
{
    "source": "test_source",
    "episode": "e01",
    "transcripts": [
        {
            "segments": [
                {
                    "text": " זהו משפט עבור",
                    "avg_logprob": -0.023
                },
                {
                    "text": "בדיקה בסיסית",
                    "avg_logprob": -0.323
                }
            ]
        },
        {
            "segments": [
                {
                    "text": "גם זה משמש ",
                    "avg_logprob": -0.063
                },
                {
                    "text": "רק לבדיקה",
                    "avg_logprob": -0.123
                }
            ]
        }
    ]
}
```
The name of the JSON file is ignored, the "source" (such as the podcast name) and "episode" fields are used to find the audio files in the audio directory (see below.)

The "segments" field is a list of segments, each segment is a dictionary with a "text" field, which is the initial transcription of the segment, and an "avg_logprob" field, which is the average log probability of the transcription. (Used to estimate the difficulty of the transcript compared to other transcripts.)

Text from all segments is joined to form the entire transcript for the specific audio file.

The ordinal index of the transcript into the "transcripts" array will be the (0-based) index of the audio file in the audio directory.

Multiple JSON files are supported - as lone as not using the "staging mode" in which case only the first found JSON is used.

# Audio File Directory

This directory is the root directory of the following structure:

```
-- Root
    -- test_source
        -- e01
            -- 0.mp3
            -- 1.mp3
            -- ...
    -- other_source
        -- e01
            -- ...
        -- e02
            -- ...
    ...
```

The "source" and "episode" fields in the JSON files are used to find the audio files in this directory.

## Local SSL support

The local web server runs with SSL support and thus requires self-signed certificate to be created.

**Note** that self-signed certificates are not trusted by browsers by default and should **not** be used in production environments.

Here is one easy way to do it if you have openssl installed:
`openssl req -x509 -newkey rsa:4096 -keyout private.key -out certchain.pem -days 365 -nodes`

Move the key/pem files to a "secrets" sub folder under the repo root.

```
mkdir -p secrets
mv private.key secrets/
mv certchain.pem secrets/
```
**note** Your browser would warn you upon first loading a page "secured" by those certificates. You could still continue though. Another way to go around this is to add the certificate to your local trusted certificates.

## Environment Configuration

Create a `.env` file in the root directory of the project. (Start off of the `.env.example` if you like)

Add the following variables:

```
DB_CONNECTION_STRING - Required
FLASK_APP_SECRET - Required (Generate Locally)
GOOGLE_CLIENT_ID - Can Be Left empty in staging mode
GOOGLE_CLIENT_SECRET - Can Be Left empty in staging mode
GOOGLE_ANALYTICS_TAG - Can Be Left empty in staging mode
FTC_USER_EMAIL - Set to some email in staging mode
FTC_STAGING_MODE - enabled staging mode (see below)
```

## Staging Mode

Setting the `FTC_STAGING_MODE=1` will enable the staging mode. This will:
- Use the FTC_USER_EMAIL email as the currently "signed in" user. (allows you to use the app without having to be logged in.)
- Will use only the first found JSON file in the transcripts dir
- Free you from setting up an OAuth client credentials for the Google IDP. (If this is not what you are working on)