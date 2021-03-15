#!/usr/bin/env groovy

node {
    checkout scm
    load('pipeline-scripts/commonlib.groovy').describeJob("tkn_sync", """
        -----------------------------------------------
        Sync the Tekton pipeline client (tkn) to mirror
        -----------------------------------------------
        http://mirror.openshift.com/pub/openshift-v4/clients/pipeline/
        Timing: Run manually by request.
    """)
}

pipeline {
    agent any
    options { disableResume() }

    parameters {
        string(
            name: 'TKN_VERSION',
            description: 'Example: 1.3.1-1',
            defaultValue: '',
            trim: true,
        )
    }

    stages {
        stage('Validate params') {
            steps {
                script {
                    if (!params.TKN_VERSION) {
                        error 'TKN_VERSION must be specified'
                    }
                    target_version = params.TKN_VERSION.split("-")[0]
                    target_dir = "/srv/pub/openshift-v4/clients/pipeline/${target_version}"
                }
            }
        }

        stage('Sync to mirror') {
            steps {
                sh "tree /mnt/redhat/staging-cds/developer/openshift-pipelines-client/${params.TKN_VERSION}/signed/all ; cat /mnt/redhat/staging-cds/developer/openshift-pipelines-client/${params.TKN_VERSION}/signed/all/sha256sum.txt"
                sshagent(['aos-cd-test']) {
                    sh "tree /mnt/redhat/staging-cds/developer/openshift-pipelines-client/${params.TKN_VERSION}/signed/all"
                    sh "cat /mnt/redhat/staging-cds/developer/openshift-pipelines-client/${params.TKN_VERSION}/signed/all/sha256sum.txt"
                    sh "echo ${target_version}"
                    sh "ssh use-mirror-upload rm --recursive --force --verbose ${target_dir}"
                    sh "ssh use-mirror-upload mkdir -p ${target_dir}"
                    sh "scp -r /mnt/redhat/staging-cds/developer/openshift-pipelines-client/${params.TKN_VERSION}/signed/all/* use-mirror-upload:${target_dir}"
                    sh "ssh use-mirror-upload ln --symbolic --force --no-dereference ${target_dir} /srv/pub/openshift-v4/clients/pipeline/latest"
                    sh "ssh use-mirror-upload /usr/local/bin/push.pub.sh openshift-v4/clients/pipeline -v"
                }
            }
        }
    }
}

def download(url) {
    sh "wget --directory-prefix ${params.TKN_VERSION} ${url}"
}
