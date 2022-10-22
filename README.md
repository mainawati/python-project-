# Python-project-
import pymysql
from flask import Flask,render_template,request,redirect,url_for
db_connection=None
db_cursor=None
app = Flask(__name__)



def db_connect():
    global db_connection,db_cursor
    try:
            db_connection = pymysql.connect(host="localhost",user="root",passwd="",database="lms",port=3306)
            print("Connected")
            db_cursor=db_connection.cursor()
            return True
    except:
        print("some error occure, cant connect to database")
        return False

def db_disconnect():
    global db_connection,db_cursor
    db_connection.close()
    db_cursor.close()

#function to fetch data from database
def getAllStudents():
    isConnected = db_connect()  
    if(isConnected):
        print("yes connected")
        getQuery = "select * from student;"  #writting query
        db_cursor.execute(getQuery)           #executing query
        allData = db_cursor.fetchall()        #fetching data from query
        #print(allData)
        db_disconnect()
        return allData

@app.route("/")
def index():
    # return "hello python"
    allData = getAllStudents()
    return render_template("index.html",data=allData)    

@app.route("/add",methods=["GET","POST"])
def addStudent():
    if request.method == "POST":

        data=request.form
        name=data["name"]
        course=data["course"]
        year=data["year"]
        contact_no=data["contact_no"]
        date=data["date"]
        
        isConnected = db_connet()
        if(isConnected):
            insertQuery = "insert into student(name,course,year,contact_no,date)values(%s,%s,%s,%s,%s);"  #writting query
            db_cursor.execute(insertQuery,(name,course,year,contact_no,date)) #executing query
            db_connection.commit()
            print("Data inserted")
            db_disconnect()
            return redirect(url_for("index"))
    return render_template("add.html")

def getStudentById(stud_id):
    isConnected = db_connect()
    if(isConnected):
        selectQuery = "select * from student where stud_id=%s;"  #writting query
        db_cursor.execute(selectQuery,(stud_id)) #executing query
        current_student=db_cursor.fetchone()
        db_connection.commit()
        db_disconnect()
        return current_student
    else:
        return False

def updateStudent(name,course,year,contact_no,date,stud_id):
    isConnected = db_connet()
    if(isConnected):
        updateQuery = "update student set name=%s,course=%s,year=%s,contact_no=%s,date=%s where stud_id=%s;"  #writting query
        db_cursor.execute(updateQuery,(name,course,year,contact_no,date,stud_id)) #executing query
        db_connection.commit()
        db_disconnect()
        return True
    else:
        return False
    
@app.route("/update",methods=["GET","POST"])
def update():
    stud_id=request.args.get("ID",type=int,default=1)
    print(stud_id)
    actual_data = getStudentById(stud_id)   
    print(actual_data)
    
    if request.method=="POST":
        data=request.form
        print("data----->",data)
        isUpdated=updateStudent(data["name"],data["course"],data["year"],data["contact_no"],data["date"],stud_id) 
        if(isUpdated): 
            return redirect(url_for("index")) 
    return render_template("update.html",data=actual_data)

def deleteStudent(stud_id):
    isConnected = db_connet()
    if(isConnected):
        deleteQuery = "delete from student where stud_id=%s ;"  #writting query
        db_cursor.execute(deleteQuery,(stud_id)) #executing query
        db_connection.commit()
        db_disconnect()
        return True
    else:
        return False

@app.route("/delete")
def delete():
   stud_id=request.args.get("ID",type=int,default=1)
   isDeleted=deleteStudent(stud_id) 
   if(isDeleted): 

    return redirect(url_for("index")) 
    return render_template("index.html")


if __name__=="__main__":
    app.run(debug=True)
