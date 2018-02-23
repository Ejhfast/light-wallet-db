# Dev Environment Setup (Mac OS X)

You may find that running `python init.py` from repo root fails or hangs, in which case you should try changing `api/app.py` line 8 from `if __name__ == "__main__":` to `if __name__ == "api.app":`. This will start the basic flask server API locally. The API relies on a local mongo database cache of the NEO blockchain to serve data to endpoints. In order to sync the blockchain with your local dev environment's mongo database, continue with the following:

## Python Version and Requirements

Make sure that you are running Python version 3.6.2 (confirm by running `python -V`).

Install dependencies by running `pip install -r requirements.txt`. Note: If you do not have `pip` installed you can install it by running `sudo easy_install pip`.

## Database Services

`mongodb, redis, memcached`
Each can be installed via homebrew:

```
brew install mongo
brew install redis
brew install memcached
```

Make sure to have each of the above services running:

```
sudo mongod
brew services start redis
brew services start memcached
```

Enter mongo shell via the `mongo` command and run the following command:

```
use neonwalletdb
db.createUser({user:'admin',pwd:'admin',roles:[{db:'neonwalletdb',role:'readWrite'}]});
db.meta.insert({"name" : "lastTrustedBlock", "value" : 1162327 });
```

Decide if you want to keep a copy of the mainnet or testnet for your local environment setup. Keep in mind that both may be excessive for your development needs. The goal of running this repo locally should be to augment the existing endpoints in a testable way.

If you wish to make a copy of the TestNet (recommended), run:
```
db.meta.insert({"name":"node_status","nodes":[{"url":"http://test1.cityofzion.io:8880","status":false,"block_height":0,"time":0},{"url":"http://test2.cityofzion.io:8880","status":false,"block_height":0,"time":0},{"url":"http://test3.cityofzion.io:8880","status":false,"block_height":0,"time":0},{"url":"http://test4.cityofzion.io:8880","status":false,"block_height":0,"time":0},{"url":"http://test5.cityofzion.io:8880","status":false,"block_height":0,"time":0}]})
```

If you wish to make a copy of the MainNet (please don't), run:
```
db.meta.insert({"name":"node_status","nodes":[{"url":"http://seed1.cityofzion.io:8080","status":false,"block_height":0,"time":0},{"url":"http://seed2.cityofzion.io:8080","status":false,"block_height":0,"time":0},{"url":"http://seed3.cityofzion.io:8080","status":false,"block_height":0,"time":0},{"url":"http://seed4.cityofzion.io:8080","status":false,"block_height":0,"time":0},{"url":"http://seed5.cityofzion.io:8080","status":false,"block_height":0,"time":0}]})
```

## Environment Variables

Add the following to your bash profile (ex. `~/.bash_profile`):

```
export MONGOUSER=admin
export MONGOPASS=admin
export MONGOURL=localhost
export MONGOAPP=neonwalletdb

export REDISTOGO_URL=localhost:6379
export MEMCACHE_SERVERS=localhost:8000

export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

The ports used for redis and memcached in the environment variables defined above are default values assigned by installation. Change them as necessary. 

Also add to your bash profile `export NET=TestNet` or `export NET=MainNet`, depending on which you chose for the previous instruction.

Don't forget to run `source ~/.bash_profile` upon saving.

## Syncing Mongo'

### Queuing Import Jobs

Run `python clock.py` from repo root. This will queue jobs through redis queue that copy blocks into your local mongodb. Wait a while until you see `repairing` followed by a final `done`, and then exit with `ctrl+c`.


### Run Import Jobs

Run `python worker.py` from repo root. This will take a while...

That's it! Your local dev env API will now serve data from your local blockchain cache stored in your local mongo database.