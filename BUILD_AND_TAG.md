## prereq
    git remote add origin_remote https://github.com/adriankumpf/teslamate.git
    branch -r

## Sync Repos
    git fetch origin_remote --tags
    git merge --allow-unrelated-histories origin_remote/master
    git push

## docker build
    docker build -t datoma/teslamate:latest .

## push to dockerhub
    docker tag datoma/teslamate:latest datoma/teslamate:1.21
    docker push datoma/teslamate:1.21
    docker push datoma/teslamate:latest

## build on Mac M1
    docker buildx build --platform linux/amd64,linux/arm64 --push -t datoma/jenkins-master:latest .
    docker buildx build --platform linux/amd64,linux/arm64 --push -t datoma/jenkins-master:1.24 .

## push to artifactory
    docker tag datoma/teslamate:latest datoma.jfrog.io/docker-local/teslamate:latest
    docker tag datoma/teslamate:latest datoma.jfrog.io/docker-local/teslamate:1.21
    docker push datoma.jfrog.io/docker-local/teslamate:1.21
    docker push datoma.jfrog.io/docker-local/teslamate:latest
