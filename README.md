# MN Pay #
This project relies on data collection and is currently not working because we don't have the data we need.
For more information about the state of this project, check out our wiki [here](https://github.com/OpenTwinCities/mnpay/wiki)!

## Getting started ##
### Dependencies ###
Your system needs the following installed:

- Docker - DigitalOcean has a [great tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04) to do this on ubuntu 16.04.
- Docker-compose - If you have pip installed this can be installed with `pip install docker-compose`

### One-time setup ###
Initialize the database and load in the data. This will be done on your local
machine outside of a docker container.
```bash
cd server
pipenv install --dev
pipenv shell
python manage.py migrate
python manage.py loadwages ../resources
```
This is creating an sqlite database in the project repo and populating it
with the data from the `resources/` directory. We do this once at the start
of the project because the application considers this database to be
a static build artifact (It has not need to mutate the data while running).

Now that this database has been built the application itself will be served
through docker.

If you run into issues, consider deleting Pipfile.lock and trying again.

### Running ###
To launch your server run
```bash
$ source ./dev_env.sh
$ docker-compose up
```
You can then direct your browser to [localhost:80](http://localhost/) to see the site.

If you are working on the server, restart the server container to see changes.
Do this by hitting `ctrl+c` in the terminal you ran `docker-compose up` in,
then run `docker-compose up` again.

### Adding server dependencies ###
If you add a dependency to the server you need to rebuild the docker containers
in order for it to get picked up. This would be the case if you ran
`pip install [anything]`. Add the requirement to `server/requirements.txt`
and then run
```bash
$ docker-compose build oneoff
$ docker-compose build web
```

### Handy commands ###
If you need to connect to a container for debugging use:
```bash
docker exec -it <container_id> /bin/bash
```

## Deployment ##
First run `docker-compose up` and `python manage.py loadwages ../resources` to
gather static files, build the client, and load in the data.

```bash
docker save -o mnpay_nginx.image mnpay_nginx
scp mnpay_nginx.image <remote_server>/<remote_path>
docker save -o mnpay_web.image mnpay_web
scp mnpay_web.image <remote_server>/<remote_path>
```

On the remote server.

```bash
docker load -i mnpay_nginx.image
docker load -i mnpay_web.image
docker-compose -f docker-compose-prod.yml up
```
