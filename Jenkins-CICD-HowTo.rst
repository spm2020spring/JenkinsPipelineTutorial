Continuous Integration and Testing Pipeline With Jenkins
==========================================================

:Date: 2021-02-07
:Authors: Hui <lanhui at zjnu.edu.cn>


We have two servers, one for hosting the remote git repository
(``git-server-ip-address``), and another (``my-server-ip-address``) for
running jenkins.  ``my-server-ip-address`` will also run our web
application, which is to be built using the jenkins pipeline.  We need to
establish the relationship between git-server-ip-address and
``my-server-ip-address`` through jenkins, so that jenkins is able to fetch
code from ``git-server-ip-address``, build it and run it.


Get the jenkins key
---------------------


``curl https://pkg.jenkins.io/debian-stable/jenkins.io.key > key``

``sudo apt-key add key``

``sudo apt-get update``



Install jenkins
--------------------

``sudo apt-get install jenkins``


Note: execute the above command under root account.




Change port
-----------------

Edit /etc/init.d/jenkins.  Find the line including 'check_tcp_port'.

``sudo nano /etc/init.d/jenkins``

Change 8080 (the default) to 8081 if 8080 is taken.

Open firewall to allow the port 8081 such that we could visit jenkins from http://my-server-ip-address:8081.

``sudo ufw allow 8081``

Turn on the jenkins service
----------------------------------

Start the jenkins service.

``sudo systemctl start jenkins``

Check status.

``sudo systemctl status jenkins``


Setup jenkins using a web browser
-------------------------------------

Visit http://my-server-ip-address:8081/.


Get the initial password for admin
--------------------------------------------

``sudo cat /var/lib/jenkins/secrets/initialAdminPassword``

Create an administrative account.


Give account name jenkins proper permission
-----------------------------------------------

Give jenkins the docker permission if it is to build a web application that uses Docker.

``sudo usermod -a -G docker jenkins``


Put Jenkinsfile in the root directory of the web app EnglishPal
-----------------------------------------------------------------

Specify branch source: ``git@git-server-ip-address:EnglishPal``.
Need to append jenkins' public ssh key to git server's key set (need some work).
Switch to jenkins account: ``sudo su jenkins``.
The public ssh key needs to be generated (by using the ssh-keygen command) and it is stored at ``/var/lib/jenkins/.ssh/id_rsa.pub``.
Append id_rsa.pub to the file ``.ssh/authorized_keys`` located at ``git-server-ip-address``.

Jenkins checks out code from a repository after each push to its
workspace (``/var/lib/jenkins/workspace``) and run the commands specified
in Jenkinsfile.


::
   
    pipeline {
    
        agent any
    
        stages {
            stage('BuildIt') {
                steps {
                    echo 'Building..'
                    sh 'sudo docker stop $(sudo docker ps -q --filter ancestor=englishpal)'
                    sh 'sudo docker build -t englishpal .'
                    sh 'sudo docker run -d -p 91:80 -v /var/lib/jenkins/workspace/EnglishPal_Pipeline_master/app/static/frequency:/app/static/frequency -t englishpal'
                    sh 'sudo docker system prune -a -f'
                }
            }
            stage('TestIt') {
                steps {
                    echo 'Testing..'
                }
            }
            stage('DeployIt') {
                steps {
                    echo 'Deploying....'
                }
            }
        }
    }
    

Now we could run the web app at ``my-server-ip-address:91``.


Limit memory usage
-------------------

Jenkins by default consumes a lot of memory space (e.g., 1GB).  To
restrict memory usage, add ``-Xmx256m`` to ``JAVA_ARGS`` in
/etc/default/jenkins and restart the jenkins service::

  sudo systemctl restart jenkins

``Xmx256m`` will restrict Java's memory consumption to 256MB.


Combined use with Selenium for automated testing
--------------------------------------------------

Tips:

- Use docker image selenium/standalone-chrome to run end to end testing without GUI.
- Must close driver after use: ``driver.quit()``.
  



References
-------------------


- https://www.cnblogs.com/xiao987334176/p/11323795.html

- https://blog.csdn.net/syc000666/article/details/104020167

- Creating your first pipeline https://www.jenkins.io/zh/doc/pipeline/tour/hello-world/

