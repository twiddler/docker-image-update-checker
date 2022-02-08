# Is my docker parent image out-of-date?

[![Test](https://github.com/twiddler/is-my-docker-parent-image-out-of-date/actions/workflows/test.yml/badge.svg)](https://github.com/twiddler/is-my-docker-parent-image-out-of-date/actions/workflows/test.yml)

Well, ask no more! This github action has the answer! :sunglasses:

Keeping your parent image up-to-date is essential to provide your built with the latest (security) patches. However, you might not want to stupidly rebuild your image everyday. Use this action to check if you really have to rebuild! :partying_face:

I haven't tested this with [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), but I guess it should be able to (only) check the last stage that really makes it into your built image.

## Inputs

| Name           | Type   | Description                                     |
| -------------- | ------ | ----------------------------------------------- |
| `parent-image` | String | The docker parent image you are building on top |
| `my-image`     | String | Your image which uses `FROM [parent-image]`     |

## Output

| Name          | Type   | Description                                                                                    |
| ------------- | ------ | ---------------------------------------------------------------------------------------------- |
| `out-of-date` | String | 'true' if `my-image` does **not** use the latest version of `parent-image`, 'false' otherwise. |

## Example

```yaml
name: check docker images

# You might want to regularly check for a new parent image release
on:
  schedule:
    - cron: "0 4 * * *"

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Check if we have to even do all this
        id: check
        uses: twiddler/is-my-docker-parent-image-out-of-date@v1
        with:
          parent-image: nginx:1.21.0
          my-image: user/app:latest

      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: user/app:latest
        # Add this line to all the steps you want to skip
        if: steps.check.outputs.out-of-date == 'true'
```
