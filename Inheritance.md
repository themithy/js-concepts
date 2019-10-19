# Prototype inheritance example
 
```
                                  getPrototypeOf/prototype 
                        Function --------------------------
                                                          |
                          ^                               |
                          |                               |
              constructor |                               |
                          |                               |
                          |                               v
                                 getPrototypeOf                         getPrototypeOf
                 function Tree  --------------->  Function.prototype  ----------------------
                                                                                           |
                          ^  |   \                                                         | 
                          |  |    \   prototype                                            |
                          |  |     ----------------                                        |
              constructor |  | new                 \                                       |
                          |  |                     |                                       |
                          |  |                     |                                       |
                          |  v                     v                                       v
             instanceOf          getPrototypeOf                    getPrototypeOf  
    Object  <-----------  tree  --------------->  Tree.prototype  --------------->  Object.prototype  ---->  null
            
                          |
                          |
                   typeof |
                          |
                          v
            
                       "object"
```
