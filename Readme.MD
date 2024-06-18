# GUVI - DAY 36

## Design database for Zen class programme.

1.  About the Task.

    > - In this task, we are given a list of entities for us to design a robust database which can help us solve the given problems.
    > - Given entities are:

    1. Users
    2. Mentors
    3. Tasks
    4. Topics
    5. Attendance
    6. Company Drives
    7. Code Kata

2.  Setup

    > - First, we create a database, using the MongoDB Compass UI.
    > - We create a database named `zenTask36` and a collection named `users` at start.
    > - Then, we create 6 more collections for our other 6 entities.

3.  Solution

    > - In this section, the query solutions for our given 6 questions are discussed. The solution snippets, with the data output are attached in the solution document. [Document URL](./GUVI%20Day%2036.pdf)

    > - The schema for the database collections are described in the schema.txt file. [Document URL](./schema.txt)

    > - The solution queries for the given 6 questions are described in the queries.txt file. [Document URL](./queries.txt)

    1.  Find all the topics and tasks which are taught in the month of October.

        ```
        db.tasks.aggregate([
          {
              $match: {
                  date: {
                      $gte: "2020-10-01",
                      $lt: "2024-11-01"
                  }
              }
          }
        ]);
        ```

    2.  Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020.

        ```
        db.company_drives.aggregate([
          {
              $match: {
                      date: {
                          $gte: "2020-10-15",
                          $lt: "2020-10-31"
                      }
                  }
          }
        ]);
        ```

    3.  Find all the company drives and students who appeared for the placement.

        ```
        db.company_drives.aggregate([
          {
              $unwind: "$attendees"
          },
          {
              $lookup: {
              from: "users",
              localField: "attendees.id",
              foreignField: "_id",
              as: "studentDetails"
              }
          },
          {
              $unwind: "$studentDetails"
          },
          {
              $group: {
                  _id: "$_id",
                  company_name: { $first: "$company_name" },
                  date: { $first: "$date" },
                  attendees: {
                      $push: {
                      id: "$attendees.id",
                      status: "$attendees.status",
                      studentDetails: "$studentDetails"
                      }
                  }
              }
          }
        ]);
        ```

    4.  Find the number of problems solved by the user in codekata.

        ```
        db.code_kata.aggregate([
          {
              $match:{
                  "user_id": ObjectId("66715716582c0c7790fa280e")
              }

          },
          {
              $project: {
                  user_id: 1,
                  problemsSolved: { $size: "$problems_solved" }
              }
          }
        ]);
        ```

    5.  Find all the mentors who have a mentee's count of more than 15.

        ```
        db.mentors.aggregate([
          {
              $project: {
                  name: 1,
                  menteeCount: { $size: "$mentees" }
              }
          },
          {
              $match: {
                  menteeCount: { $gt: 15 }
              }
          }
        ]);
        ```

    6.  Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020.

        ```
        db.attendance.aggregate([
            {
                $match:{
                    "status":{
                        $eq:"absent",
                    },
                    "date":{
                        $gte:"2020-10-15",
                        $lt:"2020-10-31"
                    }
                }
            },
            {
                $count:"totalUsers"
            }
        ]);

        db.tasks.aggregate([
            {
                $match:{
                    "submissions.status":{
                        $eq:"not submitted",
                    },
                    "date":{
                        $gte:"2020-10-15",
                        $lt:"2020-10-31"
                    }
                }
            },
            {
                $count:"totalUsers"
            }
        ]);
        ```

4.  Inference:

    > - A lot more knowledge was inferred from this task and lots of new properties and concepts were explored some of which are discussed below.

    1. aggregate:
       > - MongoDB method for performing aggregation operations on the data in a collection.
       > - It allows us to perform aggregation operations like grouping, filtering, and transforming data within a collection.
    2. $match:
       > - Query operator in MongoDB's aggregation framework which is used to filter documents that match a specific criteria.
    3. $count:
       > - Aggregation pipeline stage that returns the count of documents that pass through the pipeline.
    4. $lookup:
       > - Aggregation pipeline stage that performs a left outer join to another collection in the same database to retrieve related documents.
    5. $unwind:
       > - Aggregation pipeline stage that deconstructs an array field from the input documents to output one document for each element of the array.
    6. $group:
       > - Aggregation pipeline stage that groups documents by a specified expression and allows us to perform aggregation operations on the grouped data, such as calculating sums or averages.
    7. $project:
       > - Aggregation pipeline stage that reshapes documents by including, excluding, or adding new fields with computed values, allowing us to define the structure of the output documents.
