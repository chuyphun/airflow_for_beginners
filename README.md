## What?
This repo is a fork of
[a repo from Varya Karpenko](https://github.com/karpenkovarya/airflow_for_beginners)

I came to know of this repo because I was trying to learn Airflow through watching
YouTube videos and fell upon her
[PyConDE talk](https://www.youtube.com/watch?v=YWtfU0MQZ_4)

Because the original repo does not specify too clearly how to
reproduce everything, in particular, the `requirements.txt`'s
packages seem to conflict one another, I took the decision to
adapt it to more recent version of Airflow, i.e. version 2.6.3.


## Python Package Installation
We will be primarily using Airflow only, so there is only one
package to install. Nevertheless, we need our Airflow package
to have the ability to communicate with postgres and aws, so
we need these as providers.

Complicated said, all this could be done in the command line,
right after you've created and activated a Python virtual environment:
```shell
AIRFLOW_VERSION=2.6.3
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow[postgres,amazon]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```


## Flow of Tasks and Environment Setup
Let's recall the flow of tasks which Varya conceived:
1. Create a Postgresql table
1. Request Stack Exchange API for recent Pandas questions to insert
   into the table we just created
1. Write these questions to an AWS S3 bucket
1. There being an HTML template under `dags/email_template.html` written with `jinja`,
   we shall fill in the placeholders with data from S3 bucket
1. Send the filled HTML to the current user's mailbox

The afore-mentioned technologies do not come to our service without our
ordering them. Let's get them set up.

**N.B.**
- I personally use Linux, more precisely Arch Linux, so if the reader
  is a Windows user, there might be more work involved to adapt what is told
  here to your OS. For MacOS users, I hope it could work seamlessly
- Most of the technologies set up below are free, except AWS S3 bucket
  might cause your money if you overuse it or if you forgot to delete/turn off

1. (Free) Postgresql database.
    - It's easy to install postgresql through any Linux package manager
    - e.g. cf. <https://wiki.archlinux.org/title/PostgreSQL>
    - Along with the database, normally a new user named `postgres` will be
      created. In my case, I created another user named `phunc20` because
      `phunc20` is my husband and I was using his computer for convenience
      sake
    - The creation of postgres database and having it to work along with
      Airflow might be time-consuming if you're not familiar with them,
      but don't get intimidated. You could always refer to, say, the
      arch wiki postgres page, and debug along side with Airflow Web UI's
      log page
1. (Free) [Stack Exchange API](https://api.stackexchange.com/)
    - [Registeration](https://stackapps.com/apps/oauth/register). You will
      be asked to fill in a form, in particular,
        - Fill in the **OAuth Domain** with **`stackexchange.com`** as suggested
          in <https://stackapps.com/questions/7857/what-exactly-is-a-valid-oauth-domain-name-for-registering-your-app?rq=1>
        - **Application Website**: I don't really know what to fill in to this, but
          it kind of just works when I gave it my own personal website
        - Check the checkbox saying **Enable Client Side OAuth Flow**
        - (Optional) As a last step, I followed **the implicit OAuth 2.0 flow** on
          [this webpage](https://api.stackexchange.com/docs/authentication):
          More precisely, I typed this url into my browser
          <https://stackoverflow.com/oauth/dialog?client_id=26942&scope=no_expiry&redirect_uri=https%3a%2f%2fstackexchange.com%2foauth%2flogin_success>  
          Browser to that webpage and click on the **Approve** button
        - In order for Airflow to recognize the key, id and secret, one could
          ```shell
          export AIRFLOW_VAR_STACK_OVERFLOW_KEY=<key>
          export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_ID=<client_id>
          export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_SECRET=<client_secret>
          ```
          before one execute `airflow scheduler`
    - [View existing apps](https://stackapps.com/apps/oauth)
    - [doc](https://api.stackexchange.com/docs)
- (Free/Paid) AWS S3 Bucket
    - If you're firs time user, I remember that AWS will be free for a period
      of time, as long as your usage does not surpass some limit. But better think
      of it as a paid service, and always make sure to turn off everything you
      stop using
    - Once you sign up, just go create an S3 bucket
    - One way to allow write permission to Airflow is through creation of an
      Access key to your IAM user on AWS
- (Free) Mail
    - Any mail is supposed to be able to be configured to accomplish this step, e.g.
      Gmail, Linux system mail
    - We will be using Linux system mail
    - I did not choose to use Gmail, because, due to security reasons, one needs
      to configure quite many things in order to do so. I find it already quite
      complex for a beginner to follow thus far, so I choose to send the mail
      more easily within the Linux system
    - <https://airflow.apache.org/docs/apache-airflow/stable/howto/email-config.html>


## So, in The End, How to Run The Dag?
In one terminal (emulator),
1. Activate your python virtual environment
1. `cd` into this repo's directory and then enter
   ```shell
   export AIRFLOW_HOME=$(pwd)
   airflow webserver
   ```

In another,
1. Activate your python virtual environment
1. Also `cd` into this repo's directory and then enter
   ```shell
   export AIRFLOW_HOME=$(pwd)
   export AIRFLOW_CONN_POSTGRES_CONN_ID="postgresql://<your_username>:<user_passwd>@localhost/<your_db>"
   export AIRFLOW_VAR_STACK_OVERFLOW_QUESTION_URL="https://api.stackexchange.com/2.3/questions"
   export AIRFLOW_VAR_STACK_OVERFLOW_KEY="<your_key>"
   export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_ID="<your_client_id>"
   export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_SECRET="<your_client_secret>"
   export AIRFLOW_VAR_TAG="pandas"
   export AIRFLOW_VAR_S3_BUCKET="<your_s3_bucket_name>"
   airflow scheduler
   ```

Then go to the Airflow Web UI, trigger the dag manually, and hopefully you'll see
all that tasks succeed as I do. If all goes well, then you may check your
Linux user's mail box like this:

```shell
$ cat /var/mail/phunc20
From airflow@example.com Wed Aug  9 18:07:40 2023
Return-Path: <airflow@example.com>
Delivered-To: phunc20@beetroot
Received: from [192.168.0.134] (localhost [::1])
	by beetroot (OpenSMTPD) with ESMTP id a1a93f05
	for <phunc20@beetroot>;
	Wed, 9 Aug 2023 11:07:40 +0000 (UTC)
Content-Type: multipart/mixed; boundary="===============2197727632811132899=="
MIME-Version: 1.0
Subject: Top questions with tag 'pandas' on 2023-08-09
From: airflow@example.com
To: phunc20@beetroot
Date: Wed, 09 Aug 2023 18:07:40 +0700
Message-ID: <8a874889984854df@beetroot>

--===============2197727632811132899==
Content-Type: text/html; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: base64

PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0
PSJVVEYtOCI+CiAgICA8dGl0bGU+VGl0bGU8L3RpdGxlPgo8L2hlYWQ+Cjxib2R5Pgo8dWw+Cgog
ICA8bGk+CiAgICAgICA8YSBocmVmPSJodHRwczovL3N0YWNrb3ZlcmZsb3cuY29tL3F1ZXN0aW9u
cy83NjgyODY0NC9weXRob24tcGFuZGFzLXNlbGVjdC1hbGwtbmFuLXJvdy1hbmQtZmlsbC13aXRo
LXByZXZpb3VzLXJvdyI+UHl0aG9uIHBhbmRhcyBzZWxlY3QgYWxsIG5hbiByb3cgYW5kIGZpbGwg
d2l0aCBwcmV2aW91cyByb3c8L2E+IHRhZ2dlZDogPHN0cm9uZz5weXRob24sIHBhbmRhczwvc3Ry
b25nPgogICA8L2xpPgoKICAgPGxpPgogICAgICAgPGEgaHJlZj0iaHR0cHM6Ly9zdGFja292ZXJm
bG93LmNvbS9xdWVzdGlvbnMvNzY4MzEwNTUvaG93LXRvLWNoYW5nZS10aGUtYmFyLXdpZHRoLXdo
aWxlLWtlZXBpbmctYW4tZXZlbi1zcGFjZS1hcm91bmQtYWxsLWJhcnMiPkhvdyB0byBjaGFuZ2Ug
dGhlIGJhciB3aWR0aCB3aGlsZSBrZWVwaW5nIGFuIGV2ZW4gc3BhY2UgYXJvdW5kIGFsbCBiYXJz
PC9hPiB0YWdnZWQ6IDxzdHJvbmc+cHl0aG9uLCBwYW5kYXMsIG1hdHBsb3RsaWIsIGJhci1jaGFy
dDwvc3Ryb25nPgogICA8L2xpPgoKICAgPGxpPgogICAgICAgPGEgaHJlZj0iaHR0cHM6Ly9zdGFj
a292ZXJmbG93LmNvbS9xdWVzdGlvbnMvNzY4MzMxODIvY3JlYXRlLWEtZGF0YWZyYW1lLWZyb20t
YS1kaWN0aW9uYXJ5LXdpdGgtbGlzdC1vZi1kaWN0cy1pbi1tdWx0aXBsZS1rZXlzIj5DcmVhdGUg
YSBkYXRhZnJhbWUgZnJvbSBhIGRpY3Rpb25hcnkgd2l0aCBsaXN0IG9mIGRpY3QmIzM5O3MgaW4g
bXVsdGlwbGUga2V5czwvYT4gdGFnZ2VkOiA8c3Ryb25nPnB5dGhvbiwgcGFuZGFzPC9zdHJvbmc+
CiAgIDwvbGk+CgogICA8bGk+CiAgICAgICA8YSBocmVmPSJodHRwczovL3N0YWNrb3ZlcmZsb3cu
Y29tL3F1ZXN0aW9ucy83NjgyNzM3OC9weXRob24tYWxsLXJlc2FtcGxlLXRoZS13aG9sZS1yb3ct
bm8tbWF0dGVyLWl0LWlzLW51bGwtb3ItbnVtZXJpYy12YWx1ZSI+UHl0aG9uIGFsbCByZXNhbXBs
ZSB0aGUgd2hvbGUgcm93IG5vIG1hdHRlciBpdCBpcyBudWxsIG9yIG51bWVyaWMgdmFsdWU8L2E+
IHRhZ2dlZDogPHN0cm9uZz5weXRob24sIHBhbmRhcywgcmVzYW1wbGU8L3N0cm9uZz4KICAgPC9s
aT4KCjwvdWw+CjwvYm9keT4KPC9odG1sPg==

--===============2197727632811132899==--

From airflow@example.com Thu Aug 10 11:51:54 2023
Return-Path: <airflow@example.com>
Delivered-To: phunc20@beetroot
Received: from [192.168.1.58] (localhost [::1])
	by beetroot (OpenSMTPD) with ESMTP id ec1d32db
	for <phunc20@beetroot>;
	Thu, 10 Aug 2023 04:51:54 +0000 (UTC)
Content-Type: multipart/mixed; boundary="===============8699298428192208349=="
MIME-Version: 1.0
Subject: Top questions with tag 'pandas' on 2023-08-09
From: airflow@example.com
To: phunc20@beetroot
Date: Thu, 10 Aug 2023 11:51:54 +0700
Message-ID: <8a87488df2e56d23@beetroot>

--===============8699298428192208349==
Content-Type: text/html; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: base64

PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYWQ+CiAgICA8bWV0YSBjaGFyc2V0
PSJVVEYtOCI+CiAgICA8dGl0bGU+VGl0bGU8L3RpdGxlPgo8L2hlYWQ+Cjxib2R5Pgo8dWw+Cgog
ICA8bGk+CiAgICAgICA8YSBocmVmPSJodHRwczovL3N0YWNrb3ZlcmZsb3cuY29tL3F1ZXN0aW9u
cy83NjgzNjQ1NC9hdm9pZC1mb3ItbG9vcHMtb3Zlci1jb2x1bS12YWx1ZXMtaW4tYS1wYW5kYXMt
ZGF0YWZyYW1lLXdpdGgtYS1mdW5jdGlvbiI+QXZvaWQgZm9yIGxvb3BzIG92ZXIgY29sdW0gdmFs
dWVzIGluIGEgcGFuZGFzIGRhdGFmcmFtZSB3aXRoIGEgZnVuY3Rpb248L2E+IHRhZ2dlZDogPHN0
cm9uZz5weXRob24sIHBhbmRhcywgZGF0YWZyYW1lPC9zdHJvbmc+CiAgIDwvbGk+CgogICA8bGk+
CiAgICAgICA8YSBocmVmPSJodHRwczovL3N0YWNrb3ZlcmZsb3cuY29tL3F1ZXN0aW9ucy83Njgy
ODY0NC9weXRob24tcGFuZGFzLXNlbGVjdC1hbGwtbmFuLXJvdy1hbmQtZmlsbC13aXRoLXByZXZp
b3VzLXJvdyI+UHl0aG9uIHBhbmRhcyBzZWxlY3QgYWxsIG5hbiByb3cgYW5kIGZpbGwgd2l0aCBw
cmV2aW91cyByb3c8L2E+IHRhZ2dlZDogPHN0cm9uZz5weXRob24sIHBhbmRhczwvc3Ryb25nPgog
ICA8L2xpPgoKICAgPGxpPgogICAgICAgPGEgaHJlZj0iaHR0cHM6Ly9zdGFja292ZXJmbG93LmNv
bS9xdWVzdGlvbnMvNzY4MzEwNTUvaG93LXRvLWNoYW5nZS10aGUtYmFyLXdpZHRoLXdoaWxlLWtl
ZXBpbmctYW4tZXZlbi1zcGFjZS1hcm91bmQtYWxsLWJhcnMiPkhvdyB0byBjaGFuZ2UgdGhlIGJh
ciB3aWR0aCB3aGlsZSBrZWVwaW5nIGFuIGV2ZW4gc3BhY2UgYXJvdW5kIGFsbCBiYXJzPC9hPiB0
YWdnZWQ6IDxzdHJvbmc+cHl0aG9uLCBwYW5kYXMsIG1hdHBsb3RsaWIsIGJhci1jaGFydDwvc3Ry
b25nPgogICA8L2xpPgoKICAgPGxpPgogICAgICAgPGEgaHJlZj0iaHR0cHM6Ly9zdGFja292ZXJm
bG93LmNvbS9xdWVzdGlvbnMvNzY4NDAwMzYvaXMtdGhlcmUtYS13YXktdG8tc3VwcHJlc3MtbGVn
ZW5kLWVudHJ5LXdoZW4tcGxvdHRpbmctZGlyZWN0bHktZnJvbS1wYW5kYXMiPklzIHRoZXJlIGEg
d2F5IHRvIHN1cHByZXNzIGxlZ2VuZCBlbnRyeSB3aGVuIHBsb3R0aW5nIGRpcmVjdGx5IGZyb20g
cGFuZGFzPzwvYT4gdGFnZ2VkOiA8c3Ryb25nPnB5dGhvbiwgcGFuZGFzLCBtYXRwbG90bGliPC9z
dHJvbmc+CiAgIDwvbGk+CgogICA8bGk+CiAgICAgICA8YSBocmVmPSJodHRwczovL3N0YWNrb3Zl
cmZsb3cuY29tL3F1ZXN0aW9ucy83NjgzMzE4Mi9jcmVhdGUtYS1kYXRhZnJhbWUtZnJvbS1hLWRp
Y3Rpb25hcnktd2l0aC1saXN0LW9mLWRpY3RzLWluLW11bHRpcGxlLWtleXMiPkNyZWF0ZSBhIGRh
dGFmcmFtZSBmcm9tIGEgZGljdGlvbmFyeSB3aXRoIGxpc3Qgb2YgZGljdCYjMzk7cyBpbiBtdWx0
aXBsZSBrZXlzPC9hPiB0YWdnZWQ6IDxzdHJvbmc+cHl0aG9uLCBwYW5kYXM8L3N0cm9uZz4KICAg
PC9saT4KCiAgIDxsaT4KICAgICAgIDxhIGhyZWY9Imh0dHBzOi8vc3RhY2tvdmVyZmxvdy5jb20v
cXVlc3Rpb25zLzc2ODM4NDg3L2FyZS1udW1weS1sb2dpY2FsLWVsZW1lbnQtd2lzZS1vcGVyYXRp
b25zLWJyb2tlbi1mb3ItcGFuZGFzLTItMC1ucC1sb2dpY2FsLW9yIj5BcmUgbnVtcHkgbG9naWNh
bCBlbGVtZW50IHdpc2Ugb3BlcmF0aW9ucyBicm9rZW4gZm9yIHBhbmRhcyAyLjA/IChucC5sb2dp
Y2FsX29yKTwvYT4gdGFnZ2VkOiA8c3Ryb25nPnB5dGhvbiwgcGFuZGFzLCBudW1weTwvc3Ryb25n
PgogICA8L2xpPgoKICAgPGxpPgogICAgICAgPGEgaHJlZj0iaHR0cHM6Ly9zdGFja292ZXJmbG93
LmNvbS9xdWVzdGlvbnMvNzY4MjczNzgvcHl0aG9uLWFsbC1yZXNhbXBsZS10aGUtd2hvbGUtcm93
LW5vLW1hdHRlci1pdC1pcy1udWxsLW9yLW51bWVyaWMtdmFsdWUiPlB5dGhvbiBhbGwgcmVzYW1w
bGUgdGhlIHdob2xlIHJvdyBubyBtYXR0ZXIgaXQgaXMgbnVsbCBvciBudW1lcmljIHZhbHVlPC9h
PiB0YWdnZWQ6IDxzdHJvbmc+cHl0aG9uLCBwYW5kYXMsIHJlc2FtcGxlPC9zdHJvbmc+CiAgIDwv
bGk+Cgo8L3VsPgo8L2JvZHk+CjwvaHRtbD4=

--===============8699298428192208349==--
```

As we can see, the HTML file is being encoded in Base64. One quick way
to verify that the sending of the HTML file is safe and sound could be
as follows.
```python
In [1]: import mailbox

In [2]: mails = mailbox.mbox("/var/mail/phunc20")

In [3]: from datetime import datetime

In [4]: for mail in mails:
   ...:     if datetime.strptime(mail["Date"], "%a, %d %b %Y %H:%M:%S %z").date() == datetime.today().date():
   ...:         payload = mail.get_payload()[0].get_payload().strip()
   ...:

In [5]: payload[:50]
Out[5]: 'PCFET0NUWVBFIGh0bWw+CjxodG1sIGxhbmc9ImVuIj4KPGhlYW'

In [6]: import base64

In [7]: print(base64.b64decode(payload).decode())
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>

   <li>
       <a href="https://stackoverflow.com/questions/76836454/avoid-for-loops-over-colum-values-in-a-pandas-dataframe-with-a-function">Avoid for loops over colum values in a pandas dataframe with a function</a> tagged: <strong>python, pandas, dataframe</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76828644/python-pandas-select-all-nan-row-and-fill-with-previous-row">Python pandas select all nan row and fill with previous row</a> tagged: <strong>python, pandas</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76831055/how-to-change-the-bar-width-while-keeping-an-even-space-around-all-bars">How to change the bar width while keeping an even space around all bars</a> tagged: <strong>python, pandas, matplotlib, bar-chart</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76840036/is-there-a-way-to-suppress-legend-entry-when-plotting-directly-from-pandas">Is there a way to suppress legend entry when plotting directly from pandas?</a> tagged: <strong>python, pandas, matplotlib</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76833182/create-a-dataframe-from-a-dictionary-with-list-of-dicts-in-multiple-keys">Create a dataframe from a dictionary with list of dict&#39;s in multiple keys</a> tagged: <strong>python, pandas</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76838487/are-numpy-logical-element-wise-operations-broken-for-pandas-2-0-np-logical-or">Are numpy logical element wise operations broken for pandas 2.0? (np.logical_or)</a> tagged: <strong>python, pandas, numpy</strong>
   </li>

   <li>
       <a href="https://stackoverflow.com/questions/76827378/python-all-resample-the-whole-row-no-matter-it-is-null-or-numeric-value">Python all resample the whole row no matter it is null or numeric value</a> tagged: <strong>python, pandas, resample</strong>
   </li>

</ul>
</body>
</html>

In [8]:
```

Or if you prefer, you could also save it into a file and view the HTML file with
your browser:
```python
In [8]: html_bytes = base64.b64decode(payload)

In [9]: with open(f'{datetime.today().date()}.html', 'wb') as f:
   ...:     f.write(html_bytes)
   ...:
```
