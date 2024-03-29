# GSoC-2024-Proposal-Improve-the-Database-Cache-Backend
## Table of Contents
1. Abstract (Existing problem, Goals, Benefit)
2. High level diagram
3. Low level
4. Task breakdown
5. Reference
6. About me
## 1. Abstract
### 1.1 Existing problem
Django-Mysql has good DatabaseCache, but, the existing problem with the Django-MySQL DatabaseCache implementation lies in its **lack of compatibility** with other database backends due to differences in SQL dialects and specific features offered by each database.
Developers using databases other than MySQL may face challenges in integrating and optimizing caching solutions within their Django applications. Supporting caching across multiple database backends requires rigorous testing and maintenance to ensure compatibility, performance, and reliability.

### 1.2	Goals
First of all, we need to understanding Django-MySQL features. Do the research and document the specific features, such as ```approx_count(), _clone(), count_tries_approx()```, and other relevant methods or functionalities. And determine how these features interact with MySQL database, and how they can be utilized for caching purposes.\
Then we will design the interface that abstracts these features, and then translate the interface to different database dialects, such as PostgreSQL, SQLite, Oracle, etc. Adapt the interface and implementation to utilize the specific features and capabilities offered by each database backend.\
Then develop separate Django cache backends for each supported database backend, incorporating the translated caching functionalities. And conduct comprehensive testing to ensure that the caching functionalities work as expected across different database backends.\
Finally, document the usage and configuration of the caching solution, provide examples, practices, and troubleshooting guidelines to assist developers in effectively utilizing the caching features within their projects.

### 1.3	Benefit
Implementing a cache backend in Django for all databases can improve performance and scalability in various scenarios. Because it can reduce the load on the database server, which can lead to faster response time and optimize the performance. And this can also improve user experience. And reducing the load on the database server may resulting cost saving.


## 2. High level diagram
```
Django App            approx_count()           Database
   |                        |                   Vendor Check
   | Django ORM Queries     |                       |
   |----------------------->|                       |
   |                        |                       |
   |                        v                       |
   |                    PostgreSQL                  |
   |                      Oracle                    |
   |                      SQLite                    |
   |                        |                       |
   |    Database-specific   |                       |
   |    Approximate Count   |                       |
   |        Handling        |                       |
   |                        |                       |
   |  +--------------------+ v +-----------------+  |
   |  | PostgreSQL Approx. |   | Oracle Approx. |  |
   |  |   Count Logic      |   |  Count Logic   |  |
   |  +--------------------+   +-----------------+  |
   |                        |                       |
   |  +----------------------------------------+    |
   +->|          Database-Specific SQL         |<---+
      |              Queries                   |
      +----------------------------------------+
                   |    |    |
                   v    |    v
         +-----------------------+
         | PostgreSQL / Oracle / |
         |      SQLite DB        |
         +-----------------------+
                   |
                   v
           Approx. Count Result
```

## 3. Low level
Identify Equivalent Functionality: Research the features and capabilities of PostgreSQL, or Oracle that similar to the features provided by ‘django-mysql’.\
Implement Database Specific Functionality: Write database-specific implementations for the features provided by django-mysql. This may involve writing custom SQL queries or using specific database functions provided by PostgreSQL or Oracle.\
Conditional Logic in Code: In your Django project, conditionally execute the appropriate database-specific functionality based on the database backend being used. Django provides a ```settings.DATABASES``` setting that specifies the configuration for each database connection. You can inspect this setting to determine the database backend being used and execute the corresponding code.\
Testing and Validation: Thoroughly test your implementation to ensure that it works correctly with different database backends. Pay attention to performance considerations and ensure that the translated features behave consistently across different database systems.\
**Here is an example of how it works:**\
**This is the approx_count function in django_mysql library:**
```
def approx_count(
        self,
        fall_back: bool = True,
        return_approx_int: bool = True,
        min_size: int = 1000,
    ) -> int:
        try:
            num = approx_count(self)
        except ValueError:  # Cannot be approx-counted
            if not fall_back:
                raise ValueError("Cannot use approx_count on this queryset.")
            # Always fall through to super class
            return super().count()

        if min_size and num < min_size:
            # Always fall through to super class
            return super().count()

        if return_approx_int:
            return ApproximateInt(num)
        else:
            return num
```
**Then translate it to PostgreSQL and Oracle:**
```
def approx_count(queryset):
    if connection.vendor == 'postgresql':
        query = queryset.annotate(count=Count('*')).values('count').first()
        if query:
            return query['count'] if query['count'] is not None else 0
        else:
            return 0
    elif connection.vendor == 'oracle':
        cursor = connection.cursor()
        cursor.excute("SELECT COUNT(*) FROM (SELECT 1 FROM {} SAMPLE(10) WHERE ROWNUM <= 100000".format(queryset.model._meta.db_table))
        result = cursor.fetchone()
        return result[0] if result else 0
    else:
        # Fallback to regular count for other databases
        return queryset.count()
```

## 4. Task breakdown 
My summer break will start in the beginning of May, and end in the end of August. But, I’m thinking to take one summer class. Just one, so it won’t take me a long time to take the class, and complete the assignments. So, I will have plenty of time to studying and doing this project, especially in the weekdays. \
Before coding, I will do the research and learn the Django-mysql library. Get familiar with Django and its features. Then define the scope and requirements for this project, and go through the timelines.

### Week 1-2: Understanding Django-mysql Features
•	Research and document specific features of Django-mysql library, such as approx_count(), _clone(), count_tries_approx(), and others.\
•	Understand how these features interact with MySQL database and how they can be utilized for caching purposes.

### Week 3-4: Interface Design and Abstraction
•	Design an abstracted interface that encapsulates caching functionalities provided by Django-mysql.\
•	Determine how to translate these functionalities to different database dialects (PostgreSQL, SQLite, Oracle, etc.).

### Week 5-6: Translation to Different Database Dialects
•	Adapt the interface and implementation to utilize specific features and capabilities offered by each database backend.\
•	Begin translating the caching functionalities to PostgreSQL dialect.

### Week 7-8: Translation to Different Database Dialects (Continued)
•	Continue translating caching functionalities to SQLite dialects.\
•	Ensure compatibility and optimization for each database backend.

### Week 9-10: Translation to Different Database Dialects (Continued)
•	Continue translating caching functionalities to Oracle dialects.\
•	Ensure compatibility and optimization for each database backend.

### Week 11-12: Develope Separate Cache Backends and Testing 
•       Develop separate Django cache backends for each supported database backend.\
•	Conduct comprehensive testing to ensure caching functionalities work as expected across different database backends.\
•	Test for compatibility, performance, and reliability.\
•	Address any issues or bugs discovered during testing.

### Week 13-14: Continue Testing and Documentation
•	Continue address any issues or bugs discovered during testing.\
•	Document the usage and configuration of the caching solution.\
•	Provide examples, best practices, and troubleshooting guidelines to assist developers in effectively utilizing caching features within their projects.\
•	Prepare documentation for developers to refer to during implementation.

### Week 15: Final Review and Delivery
•	Conduct a final review of the project to ensure all objectives have been met.\
•	Prepare the final deliverables, including code repositories, documentation, and any additional resources.\
•	Deliver the completed caching solution to stakeholders.

## 5. Reference
* https://adamj.eu/tech/2015/05/17/building-a-better-databasecache-for-django-on-mysql/
* https://github.com/django/django
* https://github.com/adamchainz/django-mysql/tree/dbf40498931f2dee0cb3c20015e945ad71573c1d

## 6. About Me
My name is Qiaowen Mei, and I’m a student in Northeastern University(San Jose). I’m in UTC-7 time zone. I started use Python to programming for 1.5 years. I also program in Java, C, C++.\
My email is <u>mqiowen@gmail.com</u> . Welcome to reach me if you have any questions.
