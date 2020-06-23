import org.yaml.snakeyaml.DumperOptions
import groovy.json.JsonSlurperClassic

def wmsChart
def refsChart
def releaseVersion

pipeline {
  agent { label 'linux_onprem' }
  options {
    disableConcurrentBuilds()
  }}

  stages {
    stage('Package and publish to Artifactory') {
      when { branch 'master' }
      steps {
        script {
          sshagent(credentials: ['bitbucketUser'], ignoreMissing: true) {
            sh "git remote set-url origin ssh://git@stash.jda.com:7999/lgs-saasops/wmd-helm-charts.git"
            sh 'git fetch --tags'
            def tags = sh(returnStdout: true, script: "git tag -l | sort -V").trim().tokenize()
            if (tags.isEmpty()) {
              releaseVersion = "0.1.0"
            } else {
              def latestTagVersionParts = tags.last().tokenize('.')
              releaseVersion = "${latestTagVersionParts[0]}.${latestTagVersionParts[1]}.${latestTagVersionParts[2].toInteger() + 1}"
            }
            println "Helm chart(s) will be published with version ${releaseVersion}."
          }
        }

        withAzureKeyvault(
              azureKeyVaultSecrets: [
                    [envVariable: 'ARTIFACTORY_USER', name: 'cloud-artifactory-user', secretType: 'Secret'],
                    [envVariable: 'ARTIFACTORY_PASSWORD', name: 'cloud-artifactory-password', secretType: 'Secret']],
              keyVaultURLOverride: 'https://kv-int-eu2-shd-dev-00.vault.azure.net/',
              credentialIDOverride: 'spn-jda-cld-wmdelivery-jenkins-01-dev'
        ) {
          dir("./wms") {
            script {
              wmsChart = readYaml file: './Chart.yaml'
              sh "helm dependency build"
              sh "helm lint"
              sh "helm template ."
              sh "helm package . --version ${releaseVersion}"
              sh "curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} -X PUT https://jdasoftware.jfrog.io/jdasoftware/helm-release-local/logistics/${wmsChart.name}/${wmsChart.name}-${releaseVersion}.tgz -T ${wmsChart.name}-${releaseVersion}.tgz"
            }
          }

          dir("./refs") {
            script {
              refsChart = readYaml file: './Chart.yaml'
              sh "helm dependency build"
              sh "helm lint"
              sh "helm template ."
              sh "helm package . --version ${releaseVersion}"
              sh "curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PASSWORD} -X PUT https://jdasoftware.jfrog.io/jdasoftware/helm-release-local/logistics/${refsChart.name}/${refsChart.name}-${releaseVersion}.tgz -T ${refsChart.name}-${releaseVersion}.tgz"
            }
          }
        }
      }
    }

    stage('Promote to Dev') {
      when { branch 'master' }
      steps {
        script {
          lock("promote-lgs-dev") {
            String branch = "promote-lgs-dev-${BUILD_NUMBER}"

            sshagent(credentials: ['bitbucketUser'], ignoreMissing: true) {
              sh "git clone ssh://git@stash.jda.com:7999/lgs-saasops/lgs-dev.git"
            }
            dir("lgs-dev") {
              sh "git checkout -b ${branch}"
              writeFile(file: "env/Chart.yaml", text: promoteChart(readFile("env/Chart.yaml"), wmsChart.name, releaseVersion))
              writeFile(file: "env/Chart.yaml", text: promoteChart(readFile("env/Chart.yaml"), refsChart.name, releaseVersion))

              String configRepoCommitHash
              sshagent(credentials: ['bitbucketUser'], ignoreMissing: true) {
                sh "git add env/Chart.yaml"
                gitCommitResult = sh(returnStatus: true, script: "git commit -m \"Promote ${wmsChart.name} to ${releaseVersion}\" -m \"Promote ${refsChart.name} to ${releaseVersion}\"")
                if (gitCommitResult == 0) {
                  sh "git push -f origin ${branch}"
                  configRepoCommitHash = getCurrentCommitHash()
                  withAzureKeyvault(
                        azureKeyVaultSecrets: [
                              [envVariable: 'BITBUCKET_TOKEN_USR', name: 'bitbucket-user', secretType: 'Secret'],
                              [envVariable: 'BITBUCKET_TOKEN_PSW', name: 'bitbucket-user-pat', secretType: 'Secret']],
                        keyVaultURLOverride: 'https://kv-int-eu2-shd-dev-00.vault.azure.net/',
                        credentialIDOverride: 'spn-jda-cld-wmdelivery-jenkins-01-dev'
                  ) {
                    def pullRequestId = openPullRequest("${BITBUCKET_TOKEN_USR}", "${BITBUCKET_TOKEN_PSW}", branch, "Promote ${wmsChart.name} to ${releaseVersion}. Promote ${refsChart.name} to ${releaseVersion}")
//                  boolean isSuccessful = waitForBuilds(configRepoCommitHash, "${BITBUCKET_TOKEN_USR}", "${BITBUCKET_TOKEN_PSW}", 2)
//
//                  if (isSuccessful) {
//                    mergePullRequest("${BITBUCKET_TOKEN_USR}", "${BITBUCKET_TOKEN_PSW}", pullRequestId)
//                  } else {
//                    error "Build failed"
//                  }
//                  sh "git push --delete origin ${branch}"
                  }
                } else {
                  echo "${releaseVersion} of ${wmsChart.name} and ${releaseVersion} of ${refsChart.name}  already deployed"
                }
              }
            }
            sh "rm -rf lgs-dev"
          }
        }
      }
    }

    stage('Tag') {
      when { branch 'master' }
      steps {
        sshagent(credentials: ['bitbucketUser'], ignoreMissing: true) {
          sh "git tag -af ${releaseVersion} -m 'Release version ${releaseVersion}'"
          sh "git push origin --tags"
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}

/**
 * Promote Chart to staging env.
 *
 * @param Chart.yaml to update
 * @param Chart name to update
 * @param Chart version to rev.
 *
 * @return Updated yaml file
 */
def promoteChart(yaml, appName, newVersion) {
  def parsedYaml = load("${yaml}".toString())
  def project = parsedYaml['dependencies'].find { project ->
    project['name'] == appName.toString()
  }
  if (project) {
    project['version'] = newVersion.toString()
  } else {
    parsedYaml['dependencies'] += [name: appName.toString(), repository: '@artifactory', version: newVersion.toString()]
  }
  dump(parsedYaml)
}

static def dump(value) {
  yaml().dump(value)
}

static def load(String yamlText) {
  yaml().load(yamlText)
}

static org.yaml.snakeyaml.Yaml yaml() {
  DumperOptions options = new DumperOptions()
  options.setDefaultFlowStyle(DumperOptions.FlowStyle.BLOCK)
  new org.yaml.snakeyaml.Yaml(options)
}

/**
 * Uses the bitbucket API to open a pull request
 *
 * @param bitbucketUser used to authenticate with Bitbucket to open the pull request
 * @param bitbucketToken used to authenticate with Bitbucket to open the pull request
 * @param branch the branch to open the pull request for
 * @param title used for the description of the pull request
 *
 * @return A pull request id
 */
def openPullRequest(bitbucketUser, bitbucketToken, branch, title) {
  def jsonBody = """{"title":"${title}","description":"${title}","fromRef":{"id":"refs/heads/${branch}","repository":{"slug":"lgs-dev","name":null,"project":{"key":"lgs-saasops"}}},"toRef":{"id":"refs/heads/master","repository":{"slug":"lgs-dev","name":null,"project":{"key":"lgs-saasops"}}}}"""
  sh """curl https://stash.jda.com/rest/api/1.0/projects/lgs-saasops/repos/lgs-dev/pull-requests --silent --user ${bitbucketUser}:${bitbucketToken} --request POST --header 'Content-Type: application/json' --data '${jsonBody}' > create-pull-request-response.json"""
  String openPullRequestJson = sh(returnStdout: true, script: "cat create-pull-request-response.json")
  sh "rm create-pull-request-response.json"
  int pullRequestId = pullRequestIdFromResponseBody(openPullRequestJson)
  if (pullRequestId) {
    pullRequestId
  } else {
    error "Bad response: ${openPullRequestJson}"
  }
}

/**
 * Takes the JSON response body from calling the bitbucket API to open a pull request and parses out the id
 * @return A pull request id
 */
def pullRequestIdFromResponseBody(openPullRequestJson) {
  def parsedJson = parse(openPullRequestJson)
  if (parsedJson["id"]) {
    parsedJson["id"]
  } else if (parsedJson["errors"] && parsedJson["errors"][0] && parsedJson["errors"][0]) {
    parsedJson["errors"][0]["existingPullRequest"]["id"]
  } else {
    null
  }
}

/**
 * Takes the result of calling `getBuildStatusScript` and a number of expected builds and returns whether or not all builds have finished.
 *
 * @param buildStatusJson The output from making a request to bitbucket for build statuses
 * @param expectedBuildCount The number of builds to wait for. This is useful to make sure that you don't get a false positive if builds haven't started.
 *
 * @return boolean whether not all builds are finished
 */
def areBuildsFinished(String buildStatusJson, int expectedBuildCount) {
  List values = parse(buildStatusJson)["values"]
  values.size() >= expectedBuildCount && !values.collect { build -> build["state"] }.contains("INPROGRESS")
}

/**
 * Takes the result of calling `getBuildStatusScript` and checks that all builds are successful
 *
 * @param buildStatusJson The output from making a request to bitbucket for build statuses
 *
 * @return boolean whether not all builds are successful
 */
def areBuildsSuccessful(String buildStatusJson) {
  parse(buildStatusJson)["values"].collect { build -> build["state"] }.unique() == ["SUCCESSFUL"]
}

/**
 * Wrapper for parsing json. Gives a slightly cleaner syntax as well as making it easier to switch out the underlying implementation if needed.
 */
def parse(String json) {
  (new JsonSlurperClassic()).parseText(json)
}

/**
 * Uses Bitbucket API to wait for builds for a commit to complete
 *
 * @param commit The commit to wait for builds for
 * @param expectedBuildCount The number of builds expected. For example, for a PR that has a job for the PR and the branch, this would be 2.
 *
 * @return boolean whether or not the builds were successful. If one of more failed this will be false, otherwise true
 */
def waitForBuilds(String commit, String bitbucketUser, String bitbucketPassword, int expectedBuildCount) {
  def buildStatusJson = sh(returnStdout: true, script: "curl https://stash.jda.com/rest/build-status/latest/commits/${commit} --user ${bitbucketUser}:${bitbucketPassword} --header 'Content-Type: application/json'")
  echo formatBuildStatusResult(buildStatusJson)
  if (areBuildsFinished(buildStatusJson, expectedBuildCount)) {
    areBuildsSuccessful(buildStatusJson)
  } else {
    sleep 30
    waitForBuilds(commit, bitbucketUser, bitbucketPassword, expectedBuildCount)
  }
}

/**
 * Merges a pull request using the Bitbucket API
 * If the merge request is declined for some reason generate an error and fail the build
 */
def mergePullRequest(bitbucketUser, bitbucketToken, pullRequestId) {
  String script = """curl https://stash.jda.com/rest/api/1.0/projects/lgs-saasops/repos/lgs-dev/pull-requests/${pullRequestId}/merge?version=0 --silent --user ${bitbucketUser}:${bitbucketToken} --request POST --header 'Content-Type: application/json'"""
  String responseBody = sh(returnStdout: true, script: script)
  if (parse(responseBody).state != "MERGED") {
    echo responseBody
    error "Merge failed (see response above)"
  }
}

/**
 * @return The commit hash of the the HEAD commit of the current git repo
 */
def getCurrentCommitHash() {
  sh(returnStdout: true, script: "git rev-parse HEAD").trim()
}

/**
 * @param buildStatusJson the JSON response body from calling the bitbucket endpoint for build statuses
 *
 * @return a formatted string that's easy to read in the jenkins output
 *
 */
String formatBuildStatusResult(String buildStatusJson) {
  String format = "%-80s %-15s %s"
  String headers = sprintf(format, ["Build", "Status", "Url"])
  List rows = parse(buildStatusJson).values.collect { buildStatus ->
    sprintf(format, [buildStatus.name, buildStatus.state, buildStatus.url])
  }
  ([headers] + rows).join("\n")
}
