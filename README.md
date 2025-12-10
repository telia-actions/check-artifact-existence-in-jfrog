
# Check artifact existence in JFrog

This action is checking if some artifact exists based on given repository and search phrase in JFrog.
It is intended to be used in artifact build/packaging workflows to check if some specific artifact already exists in JFrog and to skip uploading new virtually identical artifact to JFrog.
Action gives output "artifact_exists" with possible values "True" or "False".

## Inputs

### jfrog-repo-name:
Name of JFrog repository to download from. **Required**
### jfrog-username:
  JFrog username to use for downloading artifact. Should have READ permissions. **Required**
### jfrog-password:
  JFrog user password.
### search-phrase:
  Substring of artifact name, usually short GIT SHA. Multiple values can be passed as single string delimited with '|||' (triple pipe symbols). In that case all given artifacts should exist for outcome to be 'True' **Required**
### search-phrase-cleanup:
  Flag indicating if search-phrase input should be cleaned from spaces. Accepts "True" or "False" values. "True" by default
### local-storage-path:
  Optional path of local (usually self-hosted) runner's directory where artifacts might be stored after packaging them for faster access (instead of downloading from JFrog and extracting). Action continues with searching in JFrog if no matches found locally.

## Output

### artifact-exists
Can have values "True" or "False".

## Example

```
    name: Check artifact existence in JFrog
    
    on:
    workflow_dispatch:
    
    jobs:
      check:
        outputs:
          status: ${{ steps.existance-check.outputs.artifact_exists }}
        runs-on: [ec2, windows, medium]
        name: Download and deploy artifact for some application
        steps:
          - id: existance-check
            uses: telia-actions/check-artifact-existence-in-jfrog@v1
            with:
              jfrog-repo-name: 'some-jfrog-repo-name'
              jfrog-username: ${{ vars.JFROG_USERNAME }}
              jfrog-password: ${{ secrets.JFROG_PASSWORD }}
              search-phrase: 'WEB-*-${{ github.sha }} ||| API-*-${{ github.sha }}'

      build:
        needs: check
        if: needs.check.outputs.status == 'False'
        
        steps:
           ...
```