"# zen_class_program" 

###  MongoDB Zen Class Programme Database

#### Introduction
This document outlines the schema design for a MongoDB database used to manage a Zen class programme. It includes various collections such as `users`, `codekata`, `attendance`, `topics`, `tasks`, `company_drives`, and `mentors`. Additionally, it provides MongoDB queries to retrieve specific information based on given requirements.

### Database Collections

1. **users**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "name": String,
            "email": String,
            "tasks_submitted": Boolean
        }
        ```

2. **codekata**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "user_id": ObjectId,
            "problems_solved": Number
        }
        ```

3. **attendance**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "user_id": ObjectId,
            "date": Date,
            "status": String  // "present" or "absent"
        }
        ```

4. **topics**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "topic_name": String,
            "date": Date
        }
        ```

5. **tasks**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "task_name": String,
            "date": Date
        }
        ```

6. **company_drives**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "company_name": String,
            "date": Date,
            "students_appeared": [ObjectId]
        }
        ```

7. **mentors**
    - Structure:
        ```json
        {
            "_id": ObjectId,
            "mentor_name": String,
            "mentee_ids": [ObjectId]
        }
        ```

### MongoDB Queries

1. **Find all the topics and tasks which are taught in the month of October:**
    ```js
    db.topics.find({
        "date": {
            "$gte": new Date("2020-10-01"),
            "$lte": new Date("2020-10-31")
        }
    });

    db.tasks.find({
        "date": {
            "$gte": new Date("2020-10-01"),
            "$lte": new Date("2020-10-31")
        }
    });
    ```

2. **Find all the company drives which appeared between 15-Oct-2020 and 31-Oct-2020:**
    ```js
    db.company_drives.find({
        "date": {
            "$gte": new Date("2020-10-15"),
            "$lte": new Date("2020-10-31")
        }
    });
    ```

3. **Find all the company drives and students who appeared for the placement:**
    ```js
    db.company_drives.aggregate([
        {
            "$lookup": {
                "from": "users",
                "localField": "students_appeared",
                "foreignField": "_id",
                "as": "students"
            }
        }
    ]);
    ```

4. **Find the number of problems solved by the user in codekata:**
    ```js
    db.codekata.aggregate([
        {
            "$group": {
                "_id": "$user_id",
                "total_problems_solved": { "$sum": "$problems_solved" }
            }
        }
    ]);
    ```

5. **Find all the mentors who have mentee's count more than 15:**
    ```js
    db.mentors.find({
        "$where": "this.mentee_ids.length > 15"
    });
    ```

6. **Find the number of users who are absent and tasks not submitted between 15-Oct-2020 and 31-Oct-2020:**
    ```js
    db.attendance.aggregate([
        {
            "$match": {
                "date": { "$gte": new Date("2020-10-15"), "$lte": new Date("2020-10-31") },
                "status": "absent"
            }
        },
        {
            "$lookup": {
                "from": "users",
                "localField": "user_id",
                "foreignField": "_id",
                "as": "user"
            }
        },
        {
            "$unwind": "$user"
        },
        {
            "$match": {
                "user.tasks_submitted": false
            }
        },
        {
            "$count": "absent_and_not_submitted"
        }
    ]);
    ```

### Conclusion
This README provides the structure of the collections in the Zen class programme database and MongoDB queries to perform various data retrieval tasks. The queries are designed to fetch specific information about topics, tasks, company drives, user performance, and attendance based on the requirements.