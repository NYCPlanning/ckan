# NYC Planning Labs CKAN Sandbox

CKAN is a data management system that provides a powerful platform for cataloging, storing and accessing
datasets with a rich front-end, full API (for both data and catalog), visualization
tools and more. 

NYC Planning labs is standing up a CKAN sandbox for the Planning Coordination Division to test. At this point, this repo is just a clone of the [CKAN](github.com/ckan/ckan/) repo with this `README.md` as a place to log notes about getting the sandbox up and running.

# Standing up CKAN using Docker

CKAN can be stood up quickly using three docker images, one for `solr`, one for the `postgresql` database, and one for the `CKAN` web application, [as documented here](http://docs.ckan.org/en/ckan-2.4.7/maintaining/installing/install-using-docker.html).

We experienced [this dependency issue](https://github.com/ckan/ckan/issues/3594) when running the published CKAN image (on 13 June 2017).  

The workaround was to clone the CKAN repository and add one line to the Dockerfile:
`RUN ckan-pip install --upgrade urllib3`

## Steps to get the sandbox up and running
1. Run the `solr` and `postgresql` containers from CKAN's dockerhub page
`docker run -d --name db ckan/postgresql`
`docker run -d --name solr ckan/solr`

2. SSH into the server and clone the ckan repository (in the future it will be clone this repository) 
`git clone https://github.com/ckan/ckan.git`

3. Navigate to the ckan directory you just created
`cd ckan`

4. Edit the Dockerfile

Add `RUN ckan-pip install --upgrade urllib3` after line 36.
Edit line 9 to reflect the domain that the site will be available on. `ENV CKAN_SITE_URL http://{domain}:5000`

5. Build the image
`docker build .`
This builds a new image and will yeild an `imageid` which you will need in the next two commands.

6. Create a sysadmin user named 'admin' (this creates an ephemeral container that only exists to add a new sysadmin to the database.
`docker run -i -t --link db:db --link solr:solr {imageid} /bin/bash -c '$CKAN_HOME/bin/paster --plugin=ckan sysadmin -c $CKAN_CONFIG/ckan.ini add admin'`

7. Run the CKAN container, linking to the `solr` and `postgresql` containers.
`docker run -d -p 5000:5000 --link db:db --link solr:solr {imageid}`
CKAN is hosted on port `5000` inside the container, so `-p 5000:5000` will map port `5000` on the host machine to port `5000` in the container.
You should see the CKAN site live at `http://{yourdomainorserverip}:5000`

## Notes for the future
When we're ready to set up for production, there will be an additional step of configuring Nginx on the host machine so that CKAN will be available via SSL at a custom subdomain (probably `ckan.planninglabs.nyc`or `data.planning.nyc.gov`).  The above setup should remain the same, we just need to tell Nginx to route traffic on port 443 for the custom domain to internal port `5000`.
