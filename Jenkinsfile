pipeline {
    agent any

    parameters {
        choice(
            name: 'ANY_HOSTS',
            choices: ['dev_db', 'staging_db', 'prod_db'],
            description: 'Target environment'
        )
        choice(
            name: 'DB_TYPE',
            choices: ['mysql', 'mongodb', 'postgresql'],
            description: 'Database type'
        )
        string(
            name: 'VERSION',
            defaultValue: 'latest',
            description: 'Target version (latest or specific e.g. 8.0.46-0ubuntu0.22.04.3)'
        )
    }

    stages {
        stage('Pre-Flight Check') {
            steps {
                echo "=========================================="
                echo "Target Hosts : ${params.ANY_HOSTS}"
                echo "Database Type: ${params.DB_TYPE}"
                echo "Version      : ${params.VERSION}"
                echo "=========================================="
            }
        }

        stage('Approval') {
            when {
                expression { params.ANY_HOSTS == 'prod_db' }
            }
            steps {
                input message: "Production DB patch karna hai?", ok: "Haan, karo!"
            }
        }

        stage('DB Patch') {
            steps {
                sh """
                ansible-playbook \
                    -i /home/rahulsharma/ansible-db-patch/inventory/hosts \
                    /home/rahulsharma/ansible-db-patch/db-patch.yml \
                    --limit ${params.ANY_HOSTS} \
                    -e "DB_TYPE=${params.DB_TYPE}" \
                    -e "VERSION=${params.VERSION}"
                """
            }
        }
    }

    post {
        success {
            echo "SUCCESS | ${params.DB_TYPE} patched on ${params.ANY_HOSTS}"
        }
        failure {
            echo "FAILED | ${params.DB_TYPE} patch failed on ${params.ANY_HOSTS}"
        }
    }
}

