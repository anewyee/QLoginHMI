# **Login System by Redhart** #

This is actually my first Qt 5 project driven by SQLite3.
I was basically cloning a PHP/MySQL authentication system I wrote earlier when I arrived at this idea.


This app currently has the following features:

- Registration/Login

- Profile System

- Avatar System

- Edit Profile/Password

- Delete Own Account

- Admin Panel

- Basic GUI


## **Demo** ##

Before build, you can modify the database path in 'loginsystem.cpp' or just make sure all project files are under folder 'LogSys' in Qt's default project directory.
Login as "user", "pass" for user experience or "admin", "pass" for admin rights.
The accounts are subject to modification in future so if these credentials don't match any rows in the database, browse the 'sys_users' table in the .s3db file.


## **Screenshots** ##

![19-38-50.png](https://bitbucket.org/repo/azAkE8/images/3466898737-19-38-50.png)

![19-45-43.png](https://bitbucket.org/repo/azAkE8/images/941292736-19-45-43.png)

![19-46-30.png](https://bitbucket.org/repo/azAkE8/images/1191543840-19-46-30.png)


While this app is driven by an SQLite database, it is targeted at a unified database server.
Yes, one can always serve the .s3db file over the network but why bother when MySQL basically does this?

Choice is always yours. ![E057.png](https://bitbucket.org/repo/azAkE8/images/3065839784-E057.png)


# 数据库操作

## 更新数据库

用户名存在，更新数据。

```
    QSqlQuery iQuery(db.db);
    iQuery.prepare("UPDATE sys_users SET username=(:un), passwd=(:pw), fname=(:fn), mname=(:mn), lname=(:ln), email=(:em) WHERE username=(:uno)");
    iQuery.bindValue(":un", ui->uBox_2->text());
    iQuery.bindValue(":pw", ui->pBox_2->text());
    iQuery.bindValue(":fn", ui->fBox_2->text());
    iQuery.bindValue(":mn", ui->mBox_2->text());
    iQuery.bindValue(":ln", ui->lBox_2->text());
    iQuery.bindValue(":em", ui->eBox_2->text());
    iQuery.bindValue(":uno", ui->uBox_2->text());
```

## 删除数据
删除某列不符合要求的数据

```
void LoginSystem::on_delUButton_clicked()
{
    if(QMessageBox::Yes == QMessageBox(QMessageBox::Question,
                                           "Login System", "Are you sure you want to erase all accounts?",
                                           QMessageBox::Yes|QMessageBox::No).exec())
    {
        QSqlQuery dQuery(db.db);
        dQuery.prepare("DELETE FROM sys_users WHERE rank != 0 AND rank != -1");

        if(dQuery.exec())
        {
            ui->adminLabel->setText("Query executed!");
        }
    }
}
```

## 数据重复检测

检测username 是否与填写的重复

```
QSqlQuery cQuery(db.db);
    cQuery.prepare("SELECT username FROM sys_users WHERE username = (:un)");
    cQuery.bindValue(":un", ui->uBox->text());

    if(cQuery.exec())
    {
        if(cQuery.next())
        {
            ui->uBox->setText("");
            ui->uBox->setPlaceholderText("Choose a different Username!");
            halt = true;
        }
    }
```

## 精确查找数据

![](https://i.imgur.com/ELFeKiu.png)


```
void LoginSystem::on_editButton_clicked()
{
    QSqlQuery fetcher;
    fetcher.prepare("SELECT * FROM sys_users WHERE username = (:un) AND passwd = (:pw)");
    fetcher.bindValue(":un", this->username);
    fetcher.bindValue(":pw", this->password);
    fetcher.exec();

    int idUsername = fetcher.record().indexOf("username");
    int idPasswd = fetcher.record().indexOf("passwd");
    int idEmail = fetcher.record().indexOf("email");
    int idFname = fetcher.record().indexOf("fname");
    int idMname = fetcher.record().indexOf("mname");
    int idLname = fetcher.record().indexOf("lname");

    qDebug()<<"on_editButton_clicked"<< idUsername<<idPasswd<<idEmail<<idFname<<idMname<<idLname;
    //on_editButton_clicked 1 2 7 3 4 5
    while (fetcher.next())
    {
        ui->uBox_2->setText(fetcher.value(idUsername).toString());
        ui->pBox_2->setText(fetcher.value(idPasswd).toString());
        ui->eBox_2->setText(fetcher.value(idEmail).toString());
        ui->fBox_2->setText(fetcher.value(idFname).toString());
        ui->mBox_2->setText(fetcher.value(idMname).toString());
        ui->lBox_2->setText(fetcher.value(idLname).toString());
    }

    ui->winStack->setCurrentIndex(3);
}
```

## 还原删除数据

```
void LoginSystem::on_backButton_5_clicked()
{
    this->tblMdl->revertAll();
    this->tblMdl->database().rollback();
}
```

##  确定删除

```
void LoginSystem::on_editedButton_2_clicked()
{
    if(this->tblMdl->submitAll())
    {
        this->tblMdl->database().commit();
        ui->adminLabel->setText("Saved to database!");
    }
    else
    {
        this->tblMdl->database().rollback();
    }
}

```