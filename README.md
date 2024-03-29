# Artifactory issue - npm integrity checksum failed

Steps to recreate [RTFACT-7668](https://www.jfrog.com/jira/browse/RTFACT-20509).

## Steps

1. Run `./1-artifactory-docker.sh`. This will set up a new 6.9.5 instance. And copy some required binaries to it

2. Complete the setup wizard for npm
    - browse to http://localhost:8081/artifactory and:
        - if you get an Apache Tomcat 404, run "docker restart $CONTAINER_NAME"
        - register with an Artifactory license
        - use wizard to create npm repository (should end up with one called npm-local)

3. Import

Use one of the following approaches to upload:

### 3.1 Using 'Import Repositories'
    - go to 'Admin > Import/Export > Repositories > Import'
    - select 'npm-local' under Import Repository from Path
    - browse to files and import the following, one at a time, waiting for npm index to complete between each step (can take 3 minutes, watch log)
        - `/from-npm/no-namespace/1.0.0`
        - `/from-npm/no-namespace/1.1.0`
        - `/from-npm/types-namespace/@types`
        - `/from-npm/types-namespace`
    - Under artifacts browser, right click + delete eslint-visitor-keys@1.0.0 from root (not types)
        - http://localhost:8081/artifactory/webapp/#/artifacts/browse/tree/General/npm-local/eslint-visitor-keys-1.0.0.tgz
    - Go back to 'Admin > Import/Export > Repositories > Import', reimport `/from-npm/no-namespace/1.0.0`

### 3.2 Using 'Deploy'
    - Go to 'Artifacts > Deploy'
    - Select Target Repository: 'npm-local'
    - Select Multi
    - drag on `from-npm/no-namespace/1.0.0/eslint-visitor-keys-1.0.0.tgz`
    - keep Target Path blank
    - Click 'Deploy' button
    - Right click npm-local and select Recalculate index
    - Watch logs and wait for reindex to complete
    - Do the above steps again for each of the following in turn, waiting for the reindex to complete each time:
        - `from-npm/no-namespace/1.1.0/eslint-visitor-keys-1.1.0.tgz`
        - `from-npm/types-namespace/1.0.0/eslint-visitor-keys-1.0.0.tgz` (keeping target blank)
        - `from-npm/types-namespace/1.0.0/eslint-visitor-keys-1.0.0.tgz` (setting target to /@types)
    - Under artifacts browser, right click + delete eslint-visitor-keys@1.0.0 from root (not types)
        - http://localhost:8081/artifactory/webapp/#/artifacts/browse/tree/General/npm-local/eslint-visitor-keys-1.0.0.tgz
    - Right click npm-local and select Recalculate index
    - Watch logs and wait for reindex to complete
    - Reupload `from-npm/no-namespace/1.0.0/eslint-visitor-keys-1.0.0.tgz` following the steps above (keeping target blank)

4. Replicate issue in the host
    - in the host:
        - `cd my-project`
        - Ensure local folder doesn't contain `node_modules` or `package-lock.json`
            - `rm -rf node_modules; rm package-lock.json`
        - You may need to run `npm cache clean --force` if your local npm already has correct caches of the problematic modules.
        - `npm i`

You should see

```
npm ERR! code EINTEGRITY
npm ERR! sha1-PzGA+y4pEBdxastMnW1bXDSmqB0= integrity checksum failed when using sha1: wanted sha1-PzGA+y4pEBdxastMnW1bXDSmqB0= but got sha512-OCutwjDZ4aFS6PB1UZ988C4YgwlBHJd6wCeQqaLdmadZ/7e+w79+hbMUFC1QXDNCmdyoRfAFdm0RypzwR+Qpag== sha1-HuMNeVRMqE1o1LPNsK9PIFZj3S0=. (1587 bytes)
```
