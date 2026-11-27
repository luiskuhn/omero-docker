# OMERO server with Docker Compose

This repository contains Docker Compose examples for running an OMERO server with PostgreSQL and OMERO.web. The setup is based on the [OME OMERO Docker example](https://github.com/ome/docker-example-omero) and includes an optional custom Bio-Formats image build.

## Current configuration

The Compose files on `dev` are legacy examples; their pins are not a recommendation for a new production deployment.

| Component | `docker-compose.yaml` | `custom-bio-formats/docker-compose.yaml` |
| --- | --- | --- |
| PostgreSQL | `postgres:11` | `postgres:11` |
| OMERO.server | `openmicroscopy/omero-server:5.6.7` | Built from `custom-bio-formats/server/Dockerfile` |
| OMERO.web | `openmicroscopy/omero-web-standalone:5.21.0` | `openmicroscopy/omero-web-standalone:5.21.0` |

See the [modernization roadmap](docs/modernization-roadmap.md) for the version assessment, compatibility notes, and staged upgrade plan.

## Choose a Compose file

- **Standard setup:** use the root `docker-compose.yaml`. It uses the official OMERO.server and OMERO.web images.
- **Custom Bio-Formats build:** use `custom-bio-formats/docker-compose.yaml` only when you have confirmed that you need its custom Bio-Formats version. The Dockerfile is based on OMERO.server 5.6.7 and downloads a moving upstream POM during the build, so review and qualify it before relying on it.

Do not run both Compose projects against the same data directories at the same time.

## Start the standard setup

Requirements: Docker Engine with the Docker Compose v2 plugin, and a host with the directories used for persistent data.

The Compose file binds persistent data to `/mnt/omero/database` and `/mnt/omero/omeroserver` on the Docker host. Create those directories, then start the services:

```sh
sudo mkdir -p /mnt/omero/database /mnt/omero/omeroserver
docker compose config
docker compose up -d
docker compose ps
```

Open OMERO.web at `http://<server-host>:4080`. To follow service logs:

```sh
docker compose logs -f database omeroserver omeroweb
```

Run these commands from the repository root. For the custom Bio-Formats variant, change to `custom-bio-formats/` and build the server image before starting it:

```sh
cd custom-bio-formats
docker compose build omeroserver
docker compose up -d
```

## Services, ports, and data

| Service | Host port | Persistent data |
| --- | --- | --- |
| PostgreSQL | Not published | `/mnt/omero/database` |
| OMERO.server | `4063`, `4064` | `/mnt/omero/omeroserver` |
| OMERO.web | `4080` | None configured |

The database and OMERO binary repository are bind-mounted from the host. Back up both, along with the OMERO configuration and any custom scripts, and verify a restore before upgrading. See the [OME backup and restore guide](https://omero.readthedocs.io/en/stable/sysadmins/server-backup-and-restore.html).

## Deployment and upgrade notes

- The Compose files contain the example password `omero_passwd`. Replace it before deployment; keep credentials out of source control and use your deployment's secret-management mechanism.
- Ports `4063`, `4064`, and `4080` are published on the host. Restrict access with your firewall and provide HTTPS through your institution's reverse proxy before exposing web access beyond a trusted network.
- Do not upgrade PostgreSQL by changing the image tag while reusing its existing data directory. A PostgreSQL major-version upgrade needs a planned migration, backups, and validation.
- Pin and test compatible OMERO.server, OMERO.web, and database versions together. Review the roadmap before updating these Compose files.

## Related resources

- [Modernization roadmap](docs/modernization-roadmap.md)
- [OME announcements](https://www.openmicroscopy.org/announcements/)
- [OME Docker images](https://hub.docker.com/u/openmicroscopy)
- [OME OMERO Docker example](https://github.com/ome/docker-example-omero)
- [OMERO.server Docker image](https://github.com/ome/omero-server-docker/)
- [OMERO server environment variables](https://omero.readthedocs.io/en/stable/sysadmins/unix/server-installation.html#server-env)
- [OMERO server configuration](https://omero.readthedocs.io/en/stable/sysadmins/cli/config.html)
- [Bio-Formats OMERO customization](https://github.com/sbesson/bio-formats-omero/)
