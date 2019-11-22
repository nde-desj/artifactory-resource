# Artifactory Resource

The Artifactory resource lets you integrate your pipeline jobs with maven repositories in Artifactory.

This integration allows:

- To deploy artifacts and poms generated by your build
- To retreive artifacts in a repository
- To trigger jobs when a new version of an artifact is available

## Source Configuration

- `host`: *Required.* The fully qualified domain name including the context root.
- `api_key`: *Required.* The API key of a user with sufficient read and write privileges on the repositories you wish to use
- `repository_id`: *Required.* The repository
- `group_id`: *Required.* The maven groupId of your artifact
- `artifact_id`: *Required.* The maven artifactId of your artifact

### Example

Resource Type Configuration:

``` yaml
resource_types:
  - name: artifactory
    type: docker-image
    source:
      repository: emeraldsquad/artifactory-resource
```

Resource configuration for an artifact:

``` yaml
resources:
  - name: my-artifact
    type: artifactory
    source:
      api_key: <API KEY>
      artifact_id: wonderful-artifact
      group_id: emerald.squad
      host: "https://emerald.squad.com/artifactory"
      repository_id: repo-local
```

Retreiving artifacts:

``` yaml
- get: my-artifact
```

``` yaml
- get: my-artifact
  params:
    qualifiers: [sources,javadoc]
```

Pushing local commits to the repo:

``` yaml
- put: my-artifact
  params:
    path: artifacts
```

Publish build information: _(using default build_publish params)_

``` yaml
- put: my-artifact
  params:
    path: artifacts
    upload_build: true
```

Publish build information: _(use all optional params)_

``` yaml
- put: my-artifact
  params:
    path: artifacts
    upload_build: true
    build_publish:
      build_name: wonderful-artifact-build
      git_url: <GIT URL>
      git_rev: artifacts/revision
      env_include: "BUILD|ATC"
      env_exclude: "API_KEY"
```

## Behavior

### `check`: Check for new artifacts

The resource searches for folders under `http(s)://<host>/<repository_id>/<group_id>/<artifact_id>`. It expects to get a list of valid maven versions (See <https://cwiki.apache.org/confluence/display/MAVENOLD/Versioning> for details). All subsequent versions of the given ref are returned. If no version is provided, the resource returns only the latest.

### `in`: Download the artifacts at the given ref

Download the artifacts of the given ref to the destination. It will return the same given ref as version.

#### Parameters

- `qualifiers`: *Optional.* The artifacts classifiers ex: [source,javadoc]. If specified, the resource will only retreive the qualified artifacts.

### `out`: Push the artifacts to the repository

Push the artifacts from the given path to the Artifactory maven repository. The resource will push every files presents in the folder specified in the **path** parameter. The version parameter is optionnal but the resource expect at least a version file containing a version in the format of a valid maven versions (See https://cwiki.apache.org/confluence/display/MAVENOLD/Versioning for details).

Optionally upload build information by setting the `upload_build` flag to `true`. The resource will produce a build.json file and [upload it to artifactory](https://www.jfrog.com/confluence/display/RTF/Artifactory+REST+API#ArtifactoryRESTAPI-BuildUpload). The build name will default to `<source.artifact_id>-build` and the build number will be set to [concourse's `$BUILD_ID` metadata value](https://concourse-ci.org/implementing-resource-types.html#resource-metadata).

#### Parameters

- `path`: *(required).* The path of the files to push to the repository.
- `version`: *(optional).* The path to a version file. Defaults to `<path parameter>/version`.
- `upload_build`: *(optional)* Whether or not to upload build info. Defaults to `false`.
- `build_publish`: *(optional)* Upload build info parameters.
  - `build_name` : *(optional)* The build name. Defaults to `<source.artifact_id>-build`.
  - `git_url`: *(optional)* One of:
    - The git url of the source of the artifact.
    - Path to a file containing the git url of the source of the artifact.
  - `git_rev`: *(optional)* One of:
    - The git revision of the source of the artifact.
    - Path to a file containing the git revision of the source of the artifact.
  - `env_include` : *(optional)* List of case insensitive patterns in the form of `"pattern1|pattern2|..."` Only environment variables match those patterns will be included. Default to `"."`
  - `env_exclude` : *(optional)* List of case insensitive patterns in the form of `"pattern1|pattern2|..."`. Environment variables match those patterns will be excluded. Default to `"password|pword|pwd|secret|key|token"`
