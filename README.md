## Installation
postgres, amazon


## APIs
1. [Stack Exchange API](https://api.stackexchange.com/)
    - [Registeration](https://stackapps.com/apps/oauth/register). You will
      be asked to fill in a form, in particular,
        - Fill in the **OAuth Domain** by **`stackexchange.com`** as suggested
          in <https://stackapps.com/questions/7857/what-exactly-is-a-valid-oauth-domain-name-for-registering-your-app?rq=1>
        - **Application Website**: I don't really know what to fill in to this, but
          it kind of just works when I gave it my own personal website
        - Check the checkbox saying **Enable Client Side OAuth Flow**
        - (Optional) As a last step, I followed **the implicit OAuth 2.0 flow** on
          [this webpage](https://api.stackexchange.com/docs/authentication):
          More precisely, I typed this url into my browser
          <https://stackoverflow.com/oauth/dialog?client_id=26942&scope=no_expiry&redirect_uri=https%3a%2f%2fstackexchange.com%2foauth%2flogin_success>  
          Browser to that webpage and click on the **Approve** button
        - In order for Airflow to recognize the key, id and secret, one could
          ```shell
          export AIRFLOW_VAR_STACK_OVERFLOW_KEY=<key>
          export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_ID=<client_id>
          export AIRFLOW_VAR_STACK_OVERFLOW_CLIENT_SECRET=<client_secret>
          ```
          before one execute `airflow scheduler`
    - [View existing apps](https://stackapps.com/apps/oauth)
    - [doc](https://api.stackexchange.com/docs)



## Slides and YouTube Video
This is a small example of the workflow built with Apache Airflow.

You can find slides [here](https://www.slideshare.net/varyakarpenko5/airflow-for-beginners) and watch the talk [here](https://www.youtube.com/watch?v=YWtfU0MQZ_4)

The goal is to set up a data pipline to get a fresh portion of Stack Overflow questions with tag `pandas` to our mailbox daily.

A small python script could do the job, but for the learning purposes we choose to overengineer it.

By writing this workflow we will learn the main concepts of Apache Airflow, such as:
    
* Operators
* DAG
* Tasks
* Hooks
* Variables
* Connections
* XComs

Happy learning 🤓

### Helpful resources

📝 [Apache Airflow Documentation](https://airflow.apache.org/)

#### Apache Airflow tutorials for beginners

📝 [Apache Airflow Tutorial for Data Pipelines](https://blog.godatadriven.com/practical-airflow-tutorial)

📝 [Apache Airflow for the confused](https://medium.com/nyc-planning-digital/apache-airflow-for-the-confused-b588935669df)

📝 [Airflow: Tutorial and Beginners Guide](https://www.polidea.com/blog/apache-airflow-tutorial-and-beginners-guide/)

📝 [ETL Pipelines With Airflow](http://michael-harmon.com/blog/AirflowETL.html)


#### Some more

📰 [ETL best principles](https://gtoonstra.github.io/etl-with-airflow/principles.html)

📰 [Managing Dependencies in Apache Airflow](https://www.astronomer.io/guides/managing-dependencies/)

📝 [Getting Started with Airflow Using Docker](http://www.marknagelberg.com/getting-started-with-airflow-using-docker/)

🎧 [Putting Airflow Into Production](https://overcast.fm/+H1YNx1QJE)

📝 [How to configure SMTP server for apache airflow](https://stackoverflow.com/questions/51829200/how-to-set-up-airflow-send-email)


If you have any questions or would like to get in touch with me, please drop me a message to `hello@varya.io`
