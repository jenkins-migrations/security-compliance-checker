<![CDATA[
#!/usr/bin/env groovy
// Source: https://gist.github.com/merikan/228cdb1893fca91f0663bab7b095757c (Public Gist - Educational Use)
// Test case: Complex when conditions with environment variables and expressions
pipeline {
  agent any
  
  environment {
    VALUE_ONE = '1'
    VALUE_TWO = '2'
    VALUE_THREE = '3'
  }
  
  stages {
    
    // skip a stage while creating the pipeline
    stage("A stage to be skipped") {
      when {
        expression { false } //skip this stage
      }
      steps {
        echo 'This step will never be run'
      }
    }
    
    // Execute when branch = 'master'
    stage("BASIC WHEN - Branch") {
      when {
        branch 'master'
      }
      steps {
        echo 'BASIC WHEN - Master Branch!'
      }
    }
    
    // Expression based when example with AND
    stage('WHEN EXPRESSION with AND') {
      when {
        expression {
          VALUE_ONE == '1' && VALUE_THREE == '3'
        }
      }
      steps {
        echo 'WHEN with AND expression works!'
      }
    }
    
    // Expression based when example
    stage('WHEN EXPRESSION with OR') {
      when {
        expression {
          VALUE_ONE == '1' || VALUE_THREE == '2'
        }
      }
      steps {
        echo 'WHEN with OR expression works!'
      }
    }
    
    // When - AllOf Example
    stage("AllOf") {
      when {
        allOf {
          environment name:'VALUE_ONE', value: '1'
          environment name:'VALUE_TWO', value: '2'
        }
      }
      steps {
        echo "AllOf Works!!"
      }
    }
    
    // When - Not AnyOf Example
    stage("Not AnyOf") {
      when {
        not {
          anyOf {
            branch "development"
            environment name:'VALUE_TWO', value: '4'
          }
        }
      }
      steps {
        echo "Not AnyOf - Works!"
      }
    }
  }
}
    ]]>