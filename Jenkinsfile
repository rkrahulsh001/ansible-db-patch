pipeline {
    agent any
    parameters {
        choice(name: 'ANY_HOSTS', choices: ['dev_db', 'staging_db', 'prod_db'], description: 'Target group')
        choice(name: 'DB_TYPE', choices: ['postgresql', 'mysql', 'mongodb'], description: 'Database type')
        string(name: 'VERSION', defaultValue: 'latest', description: 'Patch version')
    }
    stages {
        stage('DB Patch') {
            steps {
                sh """
                  echo "=========================================="
                  echo "Target Hosts : ${params.ANY_HOSTS}"
                  echo "Database Type: ${params.DB_TYPE}"
                  echo "Version      : ${params.VERSION}"
                  echo "=========================================="
                  ansible-playbook -i inventory/hosts db-patch.yml \
                    --limit ${params.ANY_HOSTS} \
                    -e DB_TYPE=${params.DB_TYPE} \
                    -e VERSION=${params.VERSION}
                """
            }
        }
    }
}
