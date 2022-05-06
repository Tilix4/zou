# Development environment

To start with developing on Zou you need Python 3 installed and a
Postgres database instance.

## Database

To run Postgres we recommend to use Docker (it's simpler and it won't impact
your local system):

```bash
sudo docker pull postgres
sudo docker run \
    --name postgres \
    -p 5432:5432 \
    -e POSTGRES_PASSWORD=mysecretpassword \
    -d postgres
```

## Key-value store

To run Redis we recommend to use Docker again:

```bash
sudo docker pull redis
sudo docker run \
    --name redis \
    -p 6379:6379 \
    -d redis
```

## FFMPEG

For video operations, it is required to have FFMPEG installed. For that, simply install it through your OS package manager:

```
sudo apt-get install ffmpeg
```


## Source and dependencies

Then get Zou sources:

```bash
git clone git@github.com:cgwire/zou.git
```

Install `virtualenvwrapper`:

```bash
pip install virtualenvwrapper
```

Add configuration for `virtualenvwrapper` to your .bashrc:

```bash
export WORKON_HOME=directory_for_virtualenvs
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source ~/.local/bin/virtualenvwrapper.sh
```

Create a virtual environment with `mkvirtualenv`:

```bash
mkvirtualenv zou
workon zou
```

Install dependencies:

```bash
pip install -r requirements.txt
```

## Init data

Create a database in postgres named `zoudb` with user `postgres` and password
`mysecretpassword`. Then init db:

```bash
python zou/cli.py clear-db
python zou/cli.py init-db
python zou/cli.py init-data
```

Create a first user:

```bash
python zou/cli.py create-admin super.user@mycgstudio.com --password=mysecretpassword
```

Run server:

```bash
PREVIEW_FOLDER=$PWD/previews DEBUG=1 MAIL_DEBUG=1 FLASK_DEBUG=1 FLASK_APP=zou.app python zou/debug.py
```

You can now use the API by requesting `http://localhost:5000`.


### Event server

To run the Server Events server used to update the web GUI in realtime, use the
following command.

```bash
gunicorn --worker-class geventwebsocket.gunicorn.workers.GeventWebSocketWorker -b 127.0.0.1:5001 -w 1 zou.event_stream:app
```

## Tests

To run unit tests we recommend to use another database. Following the same steps as for the main database, create a `zou_test` DB in your postgreSQL instance.

```
sudo su -l postgres
psql -c 'create database zou_test;' -U postgres
```

Once your database is created, you'll be able to perform all python tests for this dedicated database by setting the `DB_DATABASE` environment variable:

```bash
DB_DATABASE=zou_test #--code to perform--#
```

To actually run the tests, you must first init the data:

```
DB_DATABASE=zou_test zou/cli.py clear-db
DB_DATABASE=zou_test zou/cli.py init-db
DB_DATABASE=zou_test zou/cli.py init-data
```

And then perform all tests:

```
DB_DATABASE=zou_test py.test
```

In case you are writing tests for a part of the code and you don't want to run them all every time, you can specify the file to run:

```
DB_DATABASE=zou_test py.test tests/models/test_task_type.py
```

