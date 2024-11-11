NAME=reprepo
IMAGE=lcharles060/$(NAME):latest
CONTAINER=$(NAME)-run

# you can pass args like "make drun arg='...'"
arg="--help"

main:
	cat Makefile

dbuild:
	docker build . --tag $(IMAGE)

dpush:
	docker push $(IMAGE)

dpull:
	docker pull $(IMAGE)

drun:
	docker run -dt -p 8080:80 -p 2022:22 -v $(PWD)/deb_packages:/deb_packages --name $(CONTAINER) $(IMAGE)

denter:
	docker exec -it $(CONTAINER) /bin/bash

dclean:
	docker rm -f $(CONTAINER)

dpurge:
	docker image rm -f $(IMAGE) 

pclean:
	rm -rf deb_packages/*

reprepo:
	# reprepro include <os.codename> <changesfile>
	# REPREPRO_BASE_DIR=/deb_packages
	# reprepro -b $REPREPRO_BASE_DIR includedeb  <os.codename> jake_1.0-7_amd64.deb

dfresh: dclean dbuild drun denter
	
