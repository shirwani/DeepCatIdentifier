###################################################################
# PREREQ ON ALL SYSTEMS
#   Install python 3.12 if it's not already installed on the server
#   Make sure python3 is aliased to python 3.12
###################################################################

#####################################################
# SETTING UP THE LOCAL DEV MACHINE TO RUN THE SCRIPTS
#####################################################
    cd CatIdentifier
    mkdir -p ./models
    which python3 # -> shoud point to python 3.12 on the system
    python3 -m venv venv
    alias python="venv/bin/python3.12"
    alias pip="venv/bin/pip3.12"
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

######################################################
# COPYING TRAINING FILES FROM LOCAL -> TRAINING SERVER
######################################################
    ssh root@172-104-24-151.ip.linodeusercontent.com 'mkdir -p /root/CatIdentifier/datasets'
    ssh root@172-104-24-151.ip.linodeusercontent.com 'mkdir -p /root/CatIdentifier/models'
    scp ./training.py                     root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/.
    scp ./testing.py                      root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/.
    scp ./utils.py                        root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/.
    scp ./requirements.txt                root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/.
    scp ./datasets/train.h5               root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/datasets/.
    scp ./datasets/cv.h5                  root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/datasets/.
    scp ./datasets/test.h5                root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/datasets/.
    scp ./config.json                     root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/.

    ###############################
    # APPLICATION ENVIRONMENT SETUP
    ###############################
    cd ~/CatIdentifier
    apt update
    apt install python3.12-venv
    which python3 # -> shoud point to python 3.12 on the system
    python3 -m venv venv
    alias python="venv/bin/python3.12"
    alias pip="venv/bin/pip3.12"
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt

################################################
# COPYING FILES FROM LOCAL -> APPLICATION SERVER
################################################
    ssh root@66-228-35-9.ip.linodeusercontent.com 'mkdir -p /var/www/flask-apps/CatIdentifier/models'
    ssh root@66-228-35-9.ip.linodeusercontent.com 'mkdir -p /var/www/flask-apps/CatIdentifier/templates'
    ssh root@66-228-35-9.ip.linodeusercontent.com 'mkdir -p /var/www/flask-apps/CatIdentifier/static/js'
    ssh root@66-228-35-9.ip.linodeusercontent.com 'mkdir -p /var/www/flask-apps/CatIdentifier/static/images'
    scp ./application.py              root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/.
    scp ./utils.py                    root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/.
    scp ./static/js/script.js         root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/static/js/.
    scp ./templates/getUserInput.html root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/templates/.
    scp ./templates/showResult.html   root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/templates/.
    scp ./templates/imageError.html   root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/templates/.
    scp ./static/images/*             root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/static/images/
    scp ./requirements.txt            root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/.
    scp ./config.json                 root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/.

    ###############################
    # APPLICATION ENVIRONMENT SETUP
    ###############################
    cd /var/www/flask-apps/CatIdentifier
    apt update
    apt install python3.12-venv
    which python3 # -> shoud point to python 3.12 on the system
    python3 -m venv venv
    alias python="venv/bin/python3.12"
    alias pip="venv/bin/pip3.12"
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt
    pip install gunicorn

    ############################################
    # APPLICATION SERVER SETUP TO SERVER WEB APP
    ############################################
    # Update /etc/nginx/sites-available/sites on the application server
    sudo ln -s /etc/nginx/sites-available/sites /etc/nginx/sites-enabled
    sudo service nginx restart

#######################################################
# RUNNING TRAINING & APPLICATION LOCALLY ON DEV MACHINE
#######################################################
    python training.py --dev
    python application.py --dev

    ########################################
    # TO COPY MODEL FROM THE TRAINING SERVER
    ########################################
    scp "root@172-104-24-151.ip.linodeusercontent.com:/root/CatIdentifier/models/*.pkl" ./models/.

#########################################
# RUNNING TRAINING ON THE TRAINING SERVER
#########################################
    pkill -f CatIdentifier
    nohup python training.py > /dev/null 2>&1 &

    ##########################################################################
    # COPYING THE TRAINED MODEL FROM THE TRAINING SERVER -> APPLICATION SERVER
    ##########################################################################
    cd ~/CatIdentifier
    scp ./models/* root@66-228-35-9.ip.linodeusercontent.com:/var/www/flask-apps/CatIdentifier/models/.

###################################################
# RUNNING THE APPLICATION ON THE APPLICATION SERVER
###################################################
    cd /var/www/flask-apps/CatIdentifier
    pkill -f CatIdentifier

    #################################
    # To use model from ./config.json
    #################################
    nohup gunicorn -b 127.0.0.1:5004 application:app > /dev/null 2>&1 &

    ##########################################
    # To use model specified from command line
    ##########################################
    # MODEL="m_20250308004639.pkl" nohup gunicorn -b 127.0.0.1:5004 application:app > /dev/null 2>&1 &
