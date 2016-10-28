# Motivation

Have you found and then lost a favorite performance on YouTube because of the following message?
- "One or more videos have been removed for your playlist, because they have been deleted from YouTube."

This simple script lets you archive your YouTube playlists as MP3 files:
- The MP3 files are stored in an Amazon S3 bucket for private use.
- You can easily transfer them to an SD card or USB stick, in order to listen them in your car or your mobile phone.
- You can have an automated daily synchronization with your YouTube playlists, if you execute this script daily from a server.
- You could listen to your MP3 archive online directly from your S3 bucket. I haven't tried this but it looks doable.

# Prerequisites

Install all dependencies. You don't need root access.

Additionally all dependencies are self-contained which lets you run this on any Linux installation like a server, container, shared-hosting account, AWS Lambda, etc.

```bash
cd ~/bin   # make sure this path is in included in your $PATH locations

# youtube-dl: https://rg3.github.io/youtube-dl/download.html

wget https://yt-dl.org/downloads/latest/youtube-dl
chmod 755 youtube-dl

# Libav Static Builds: https://johnvansickle.com/libav/

if [ "$(uname -m)" == 'x86_64' ]; then ARCH=64 ; else ARCH=32 ; fi
wget "https://johnvansickle.com/libav/builds/libav-git-${ARCH}bit-static.tar.xz"
tar -xf "libav-git-${ARCH}bit-static.tar.xz"
rm "libav-git-${ARCH}bit-static.tar.xz"
ln -s libav-git-*/{avconv,avprobe} .     # there must be only one directory "libav-git-*"

# AWS CLI bundle: http://docs.aws.amazon.com/cli/latest/userguide/installing.html#install-bundle-other-os

wget https://s3.amazonaws.com/aws-cli/awscli-bundle.zip
unzip awscli-bundle.zip
./awscli-bundle/install -i "$(pwd)/aws-lib" -b "$(pwd)/aws"
rm -r awscli-bundle awscli-bundle.zip

# mp3gain

wget "https://github.com/famzah/mp3gain-static/raw/master/mp3gain-${ARCH}bit-static.tar.xz"
tar -xf "mp3gain-${ARCH}bit-static.tar.xz"
rm "mp3gain-${ARCH}bit-static.tar.xz"
mv "mp3gain-${ARCH}bit-static" mp3gain

# youtube-mp3-archive

wget 'https://raw.githubusercontent.com/famzah/youtube-mp3-archive/master/youtube-to-s3'
chmod 755 youtube-to-s3
```

# AWS credentials

Create a new AWS user or use an existing one.

Grant this user access to your S3 bucket for mp3 archives. Don't create the S3 bucket, yet.

Here is a sample S3 policy for your new user:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1477505415000",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::youtube-mp3.famzah",
                "arn:aws:s3:::youtube-mp3.famzah/*"
            ]
        }
    ]
}
```

Then continue with the setup:

```bash
# configure your AWS user credentials

aws configure

# configure your S3 bucket info

echo "BUCKET_LOCATION=eu-central-1" > ~/.aws/mp3-bucket.conf
echo "BUCKET_NAME=youtube-mp3.famzah" >> ~/.aws/mp3-bucket.conf

# create the S3 bucket

source ~/.aws/mp3-bucket.conf
aws s3api create-bucket --create-bucket-configuration LocationConstraint="$BUCKET_LOCATION" --acl private --bucket "$BUCKET_NAME"
```

# YouTube Playlists

Configure all your YouTube playlists that you want to mirror as mp3 files.

```bash
# Create a sample playlists config file.
# Each line consists of the YouTube playlist short-URL, space,
# and then the destination folder in S3 to save the mp3 files.

echo 'PL4B3A429C539692A2 rock/evergreens' > playlists.conf

# upload the new playlists config to your S3 bucket

source ~/.aws/mp3-bucket.conf
aws s3 cp playlists.conf s3://$BUCKET_NAME/playlists.conf
```
