# Debugging

- [Debugging](#debugging)
  - [Debugging the DTIAS Installer](#debugging-the-dtias-installer)

## Debugging the DTIAS Installer

The easiest way to start debugging the installer is to run the dtias-installer container manually.

```bash
docker run --network host -h dtias-installer --name dtias-installer \
-v /home/grant/DTIAS-bundle-v2.1.0/license:/installer/license \
-v /home/grant/DTIAS-bundle-v2.1.0/dtias_config.yaml:/installer/dtias_config.yaml \
-v /home/grant/DTIAS-bundle-v2.1.0/flcm-telemetry-app-v2.1.0-rc.17.tar.gz:/installer/telemetry \
-v /home/grant/.docker/:/tmp/ \
-it dtias-installer:v2.1.0 /bin/bash
```

You can replace the `install` section in the Makefile with the following to get the installer to print out the values it is running with Docker:

```bash
install: check_if_install_target_in_accepted_values check_if_skip_checks_in_accepted_values load clean-container
	@if [ "$$INSTALL_TARGET" != "RHOCP" ]; then \
		make configure_docker; \
	fi
	@if [ "$$INSTALL_TARGET" == "PREREQUISITE" ]; then \
		echo "docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		-v $(DOCKER_TLS_DIR):/tmp/ \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) \"/installer/startup.sh\" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/pre-check-$(DATE_CMD).log"; \
		docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		-v $(DOCKER_TLS_DIR):/tmp/ \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) "/installer/startup.sh" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/pre-check-$(DATE_CMD).log; \
	elif [ "$$INSTALL_TARGET" == "RHOCP" ]; then \
		echo "docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) \"/installer/startup.sh\" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/install-$(DATE_CMD).log"; \
		docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) "/installer/startup.sh" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/install-$(DATE_CMD).log; \
	else \
		echo "INSTALL_TARGET=$$INSTALL_TARGET"; \
		echo "SKIP_CHECK=$$SKIP_CHECK"; \
		echo "BMP_SLIM_DEPLOYMENT=$$BMP_SLIM_DEPLOYMENT"; \
		echo "DTIAS_VERSION=$$DTIAS_VERSION"; \
		echo "VERBOSE=$$VERBOSE"; \
		echo "docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		-v $(shell pwd)/$(TELE_BUNDLE):/installer/$(TELE_FOLDER) \
		-v $(DOCKER_TLS_DIR):/tmp/ \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) \"/installer/startup.sh\" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION None $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/install-$(DATE_CMD).log"; \
		docker run --network host -h $(CONTAINER_NAME) --name $(CONTAINER_NAME) \
		-v $(shell pwd)/$(LICENSE_FOLDER):/installer/$(LICENSE_FOLDER) \
		-v $(shell pwd)/$(MCLOUD_FILE):/installer/$(MCLOUD_FILE) \
		-v $(shell pwd)/$(TELE_BUNDLE):/installer/$(TELE_FOLDER) \
		-v $(DOCKER_TLS_DIR):/tmp/ \
		$(INSTALLER_IMAGE):$(INSTALLER_VERSION) "/installer/startup.sh" $$INSTALL_TARGET $$SKIP_CHECK $$BMP_SLIM_DEPLOYMENT $$DTIAS_VERSION None $$VERBOSE | tee $(LOG_DIRECTORY_PATH)/install-$(DATE_CMD).log; \
	fi
```

- Once you drop into the installer, it runs `/installer/startup.sh` which then spawns `bootstrap.sh`. In my case it ran with these arguments `/installer/scripts/bootstrap.sh DTIAS2.1 false false DTIAS2.1 None`