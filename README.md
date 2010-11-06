# mongodb-session

Mongodb-session use mongodb as a Clojure/Ring's http session storage.

## Usage
### Functional access
**ring.middleware.session** version. Following examples are originated from [sandbar-examples](https://github.com/brentonashworth/sandbar-examples/blob/master/sessions/src/sandbar/examples/session_demo.clj).<br>
Ring-core must be higher than 0.3.0, because mongodb-session depend on the protocol defined ring.middleware.session.store.
    (ns hello
      (:require [ring.middleware.session :as rs]
                [somnium.congomongo :as mongo]
                [hozumi.mongodb-session :as mongoss]
	        ...))

    (mongo/mongo! :db "mydb" :host "127.0.0.1")

    (defroutes my-routes ....)
    
    (def app (-> my-routes
               (rs/wrap-session {:store (mongoss/mongodb-store)})))

Then, you can use mongodb session in the same way as in-memory one.
    (defn functional-handler
      "Functional style of working with a session."
      [request]
      (let [counter (if-let [counter (-> request :session :counter)]
                      (+ counter 1)
                      1)]
        {:status 200
         :headers {"Content-Type" "text/html"}
         :body (html
                (layout "Functional" counter (link-to "/stateful" "Stateful")))
         :session {:counter counter}}))

Let's look at the mongodb.
    % bin/mongo
    MongoDB shell version: 1.6.3
    connecting to: test
    >
    > use mydb
    switched to db mydb
    > db.ring_sessions.find()
    { "_id" : "0a7047f6-ad8a-45b0-b214-ba18830b9851",
      "_date" : "Sat Nov 06 2010 08:33:58 GMT+0900 (JST)",
      "counter" : 3 }
**_id** means cookie value of ring-session.<br>
**_date** means when this session is started.<br>
Default collection name mongodb-session use is **ring_sessions**. You can change this default collection name like below.
    (def app (-> my-routes
               (rs/wrap-session {:store (mongoss/mongodb-store :my_sessions)})))

### Statuful access
[sandbar.stateful-session](https://github.com/brentonashworth/sandbar) version.
    (ns hello
      (:require [sandbar.stateful-session :as stateful]
                [somnium.congomongo :as mongo]
                [hozumi.mongodb-session :as mongoss]))
	    
    (mongo/mongo! :db "mydb" :host "127.0.0.1")
    
    (defroutes my-routes ....)

    (def app (-> my-routes
               (stateful/wrap-stateful-session {:store (mongoss/mongodb-store)})))
You don't need to include :session entry in the response map.
    (defn stateful-handler
      "Stateful style of working with a session."
      []
      (let [counter (+ 1 (stateful/session-get :counter 0))]
        (do (stateful/session-put! :counter counter)
            (html
                 (layout "Stateful" counter (link-to "/functional" "Functional"))))))

mongodb
    > db.ring_sessions.find()
    { "_id" : "0a7047f6-ad8a-45b0-b214-ba18830b9851",
      "_date" : "Sat Nov 06 2010 08:33:58 GMT+0900 (JST)",
      "sandbar.stateful-session/session" : { "counter" : 2 } }

## Installation
Leiningen
    [org.clojars.hozumi/mongodb-session "1.0.0-SNAPSHOT"]

