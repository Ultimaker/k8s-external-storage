#!/usr/bin/groovy
@Library('k8s-jenkins-tools') _
import com.ultimaker.Slug
import com.ultimaker.Random

def credentialsFile = 'gcloud-jenkins-service-account'
def projectName = 'um-website-193311'

def validFor = '4 days'

def slugify = new Slug()
def branchSlug = slugify.slug(env.BRANCH_NAME)

def random = new Random()
def podLabel = random.string(32)

podTemplate(label: "${podLabel}", inheritFrom: 'default') {
  node("${podLabel}") {
    def scmVars = checkout scm
    def commitHash = scmVars.GIT_COMMIT

    stage('authenticate gcloud') {
      container('jnlp') {
        withCredentials([file(credentialsId: credentialsFile, variable: 'GCLOUD_KEY_FILE')]) {
          sh "gcloud auth activate-service-account --key-file=${GCLOUD_KEY_FILE}"
          sh "gcloud config set project ${projectName}"
        }
      }
    }

    stage('create images') {
      parallel(
        "snapshot-controller": {
          stage('build') {
            sh "docker build -t eu.gcr.io/${projectName}/external-storage/snapshot-controller:${branchSlug} ./snapshot-controller/"
          }
          stage('push branch slug') {
            sh "gcloud docker -- push eu.gcr.io/${projectName}/external-storage/snapshot-controller:${branchSlug}"
          }
        },
        "snapshot-provisioner": {
          stage('build') {
            sh "docker build -t eu.gcr.io/${projectName}/external-storage/snapshot-provisioner:${branchSlug} ./snapshot-provisioner/"
          }
          stage('push branch slug') {
            sh "gcloud docker -- push eu.gcr.io/${projectName}/external-storage/snapshot-provisioner:${branchSlug}"
          }
        }
      )
    }

    stage('notify') {
        slackSend color: 'good',
            channel: "#um_com_deployments",
            message: """
A new version of kubernetes-incubator/external-storage has been built (${branchSlug}/${commitHash}).
  """
    }
  }
}
