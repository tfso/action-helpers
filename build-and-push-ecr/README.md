# Build and Push ECR

This action is used to build a docker image and push it to AWS ECR. This image will then be available throughout the 24SO AWS organization.

The image that is built will get the following URI (which is available as an output of the action `image_uri`). `${{ env.CENTRAL_ECR_REPOSITORY_ACCOUNT_ID }}.dkr.ecr.eu-west-1.amazonaws.com/${{ inputs.workload_name }}//${{ inputs.image_name }}:<review | prod>-${{ inputs.tag }}`. Which means that the tag will be end up being `<review | prod>-${{ inputs.tag }}`. Whether it is `prod-` or `review-` is determined by the `review` flag, if set to `false` (default), it will be `prod-`. Ensure that production workload _only_ use `prod-` images, as `review-` images are deleted after a fortnight.

It is important that the `workload_name` matches your actual workload, if it does not, the deployment role will not be allowed to push the image. If you're unsure about the name, just ask someone from Team AWSome.

Example usage for a repository that has a docker file under a folder called **api**.
```yaml
name: ECR Push

on:
  push:
    branches:
      - "**"

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    environment: dev
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Build and Push to ECR
        id: ecr
        uses: tfso/action-helpers/build-and-push-ecr@v1
        with:
          ecr_account_id: ${{ env.CENTRAL_ECR_REPOSITORY_ACCOUNT_ID }}
          workload_name: apiworker-example
          image_name: api-example
          tag: v1.0.4
          team: awsome
          service: example.tfso.io
          file: api/Dockerfile
          # review: true # This should be true on branches/prerelease

      - name: Display Outputs
        run: |
          echo "Image URI: ${{ steps.ecr.outputs.image_uri }}"

```