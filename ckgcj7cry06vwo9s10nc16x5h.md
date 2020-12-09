## Docker Time Sync Agent - OS X

[docker-time-sync-agent](https://github.com/arunvelsriram/docker-time-sync-agent) is a tool to overcome the time drift issue in [Docker for Mac's](https://docs.docker.com/docker-for-mac/) VM that runs the Docker Engine. This issue mostly occurs when Mac wakes up after a long sleep. Docker daemon fails to update the VM's time after the computer wakes up from sleep. The result is that VM's clock will be set to a past time. This in turn will make Docker containers use that time.

So what is the problem if the containers use a wrong time ? Well, some services (like S3, Okta) will block requests originating from a source whose time is wrong. Quoting few troubles I have faced due to this time drift:

* Failing file uploads to AWS S3 (says 403 Forbidden)
* Unsuccessful SAML Authentication in Okta

I face this issue at least once a day as I close my laptop's lid when going for a break or lunch.

docker-time-sync-agent listens to system wakeup event and runs a shell script ([update-docker-time](https://github.com/arunvelsriram/docker-time-sync-agent/blob/master/docker-time-sync-agent/update-docker-time)) on occurrence of such event. This shell script in turn will update the VM's time. Using [launchd](https://www.launchd.info/), docker-time-sync-agent can be made to autostart during login, so that on every wakeup time sync happens automatically. This setup is provided through the installation script provided in the GitHub repo.

From [Docker forum](https://forums.docker.com/t/time-in-container-is-out-of-sync/16566) and [GitHub issues](https://github.com/docker/for-mac/issues/17) page it seems like this problem will be fixed in the upcoming releases. But for now this is a workaround. Hope this is helpful.

#### Links:

* [https://github.com/arunvelsriram/docker-time-sync-agent](https://github.com/arunvelsriram/docker-time-sync-agent)
* [https://docs.docker.com/docker-for-mac/](https://docs.docker.com/docker-for-mac/)
* [http://www.launchd.info/](http://www.launchd.info/)
* [https://forums.docker.com/t/time-in-container-is-out-of-sync/16566](https://forums.docker.com/t/time-in-container-is-out-of-sync/16566)
* [https://github.com/docker/for-mac/issues/17](https://github.com/docker/for-mac/issues/17)
