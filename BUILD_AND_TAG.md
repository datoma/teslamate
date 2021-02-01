docker build -t teslamate .

docker tag teslamate:latest datoma/teslamate:latest
docker tag teslamate:latest datoma/teslamate:1.21
docker push datoma/teslamate:1.21
docker push datoma/teslamate:latest


docker tag teslamate:latest datoma.jfrog.io/docker-local/teslamate:latest
docker tag teslamate:latest datoma.jfrog.io/docker-local/teslamate:1.21
docker push datoma.jfrog.io/docker-local/teslamate:1.21
docker push datoma.jfrog.io/docker-local/teslamate:latest
