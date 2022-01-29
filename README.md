# learning-docker

Just some random notes on using Docker and building Docker images.

## Scenario

- Need to create an image containing a cpp project build
- The image shall contain only the dependencies, libraries and binaries necessary for execution
- The image creation shall utilize caches rather than install dependencies / build from scratch each time
- Dependencies: debian, python and conan packages
- Binaries: shall be in directory /release
- On local machine we speed up building the project by utilizing ccache directory: /ccache

### Current approach
  - Image creation pipeline: 
    - Check out code, build binaries utilizing ccache on host
    - Dockerfile: `RUN apt install dependency-packages`
    - Last Dockerfile instruction: `COPY ./built-project /release`
    - Assumes:
      Steps for checkout, build and create-image on same host
      Building as same user that is the user in the image 

### Downsides
  - Building binaries locally:
    - Some files in /release contain paths referencing $HOME: different for each user creating the image
    - => Image is only reproducible if built by a pipeline
    
### Ideally
  - We'd have all steps leading to final image inside the Dockerfile and not compromise image creation duration

## Solution

### BuildKit features allowing various caching possibilities
- `--mount=type=tmpfs` for ramdisk [Link to StackOverflow](https://stackoverflow.com/questions/54638475/build-a-docker-image-using-docker-build-and-a-tmpfs)
- `--mount=type=cache` for caching dependency packages  [Link to BuildKit docu](https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#example-cache-apt-packages)
