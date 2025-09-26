#!/bin/bash

USERID=$(id -u)
R="\e[31m"
G="\e[32m"
Y="\e[33m"
W="\e[0m"

LOG_FOLDER="/var/log/shell-roboshop"
SCRIPT_NAME="$(echo $0 | cut -d '.' -f1)"
LOG_FILE="$LOG_FOLDER/$SCRIPT_NAME.log"

mkdir -p $LOG_FOLDER
echo -e "Script started executing at: $(date)" | tee -a $LOG_FILE

if [ $USERID -ne 0 ]; then
    echo -e "ERROR::Please run this with root ccess" | tee -a $LOG_FILE
    exit 1
 fi


SCRIPT_PATH=$PWD

dnf module disable nodejs -y &>>$LOG_FILE

dnf module enable nodejs:20 -y &>>$LOG_FILE

dnf install nodejs -y &>>$LOG_FILE

id roboshop &>>$LOG_FILE
if [ $? -ne 0 ]; then
    useradd --system --home /app --shell /sbin/nologin --comment "roboshop system user" roboshop &>>$LOG_FILE
else
    echo -e "User already exists...$Y SKIPPING $W" | tee -a &>>$LOG_FILE
fi

mkdir -p  /app &>>$LOG_FILE

rm -rf /tmp/catalogue.zip &>>$LOG_FILE

curl -o /tmp/catalogue.zip https://roboshop-artifacts.s3.amazonaws.com/catalogue-v3.zip  &>>$LOG_FILE

cd /app &>>$LOG_FILE

rm -rf /app/* &>>$LOG_FILE

unzip /tmp/catalogue.zip &>>$LOG_FILE

npm install  &>>$LOG_FILE

cp $SCRIPT_PATH/catalogue.service  /etc/systemd/system/catalogue.service &>>$LOG_FILE

systemctl daemon-reload &>>$LOG_FILE

systemctl enable catalogue &>>$LOG_FILE

systemctl start catalogue &>>$LOG_FILE

cp $SCRIPT_PATH/mongo.repo /etc/yum.repos.d/mongo.repo &>>$LOG_FILE

dnf install mongodb-mongoshdffg -y &>>$LOG_FILE

MONGODATA=$(mongosh mongo.mkreddy.shop --quiet --eval "db.getMongo().getDBNames().indexOf('catalogue')" )
if [ $MONGODATA -le 0 ]; then
    mongosh --host mongo.mkreddy.shop </app/db/master-data.js &>>$LOG_FILE
else
  echo -e "Master data already loaded.. $Y SKIPPING $W"
fi

systemctl restart catalogue &>>$LOG_FILE




