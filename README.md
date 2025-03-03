# clojurians-slack

Some of my contributions to [Clojurians Slack](http://clojurians.net/).

Each solution is also available at [Clojurians Slack Archive](https://clojurians-log.clojureverse.org/).

## Data Transformations

### Transform vector

Task:

  `I have a vector of timestamp [1600 1601 1602 1603] and another matrix containing some value [ [1600 1] [1603 3]] . I want final result like [[1600 1] "N/A" "NA " [1603 3]] `

[My solution](https://clojurians-log.clojureverse.org/beginners/2021-11-30/1638266313.001700):
```
(defn my-fn [v matrix]
  (let [map1 (into {} matrix)]
    (mapv #(if-let [ret (find map1 %)] ret "N/A") v)))

(my-fn [1600 1601 1602 1603] [[1600 1][1603 3]])
=> [[1600 1] "N/A" "N/A" [1603 3]]
```

### Group by second, return only values
  
Example:
`([1 0] [2 1] [3 0] [4 1] [5 0] [6 1]) to  ((1 3 5) (2 4 6))`

[My solution](https://clojurians-log.clojureverse.org/beginners/2021-08-27/1630057186.058500):
```
(->> '([1 0] [2 1] [3 0] [4 1] [5 0] [6 1])
     (group-by second)
     (map val)
     (map #(map first %)))
```

### Select keys:
  
Example:
```
 (foo {:a {:b 1}
        :c 2
        :d {:e {:f 3 :g 4}}}
       [[:a :b]
        [:d :e :f]])
  => {:a {:b 1}
      :d {:e {:f 3}}}
```
[My solution](https://clojurians-log.clojureverse.org/beginners/2021-11-29/1638202974.423600):
```
(defn deep-select-keys [map1 keyseq]
  (reduce #(assoc-in %1 %2 (get-in map1 %2)) {} keyseq))
```

### [Sum over columns](https://clojurians-log.clojureverse.org/beginners/2021-11-30/1638286865.022700)
```
(apply mapv + [[5 3 2 ] [1 2 5] [0 1 3]])
=> [6 6 10]
```

### Filter keys by namespace

Task:
```
Is there a good way, to obtain part of a map, via qualified key (namespace) , to the tune of...
(select-keys {:x/a 2 :x/b 3 :y/a 1} [:x/*])
Obtaining a particular part of the map by namespace.
```

[My solution](https://clojurians-log.clojureverse.org/beginners/2021-12-19/1639956153.223600):
```
(->> {:x/a 2 :x/b 3 :y/a 1}
     (filter (fn [[k v]] (re-matches #":x/.*" (str k))))
     (into {}))
=> #:x{:a 2, :b 3}
```
Better one is: `(= (namespace k) "x")`

### Remove namespace

Task:
```
For example, I have a map like {:myapp/a 1 :myapp/b 2} and I want to get {:a 1 :b 2}
```
[My solution](https://clojurians-log.clojureverse.org/beginners/2022-03-07/1646635599.183219):
```
(-> {:myapp/a 1 :myapp/b 2}
    (update-keys #(keyword (name %))))
```

## Juxt

### [Juxt usage](https://clojurians-log.clojureverse.org/beginners/2021-12-02/1638479674.293500)
```
(let [[x y] ((juxt min max) 1 2 3 4)]
  (println x y))
1 4
=> nil
```

## Time

### Weekly dates

Task:
```
is there any way to generate the weekly dates from 2017-2021. For example today is thursday how to generate dates of Thursdays from 2017-2021 ?
```

[My solution](https://clojurians-log.clojureverse.org/beginners/2021-12-02/1638461416.217200):
```
(import java.time.LocalDate)

(->> (LocalDate/parse "2017-01-05")
     (iterate #(.plusDays % 7))
     (take-while #(> 2022 (.getYear %))))
```

## Strings, output, printing

* [Interleave string with newline](https://clojurians-log.clojureverse.org/beginners/2021-07-11/1626013784.343500)
```
(clojure.string/join "\n" "This is text")
(clojure.pprint/cl-format false "~{~a~^~%~}" "This is text")
```

* [Escape newlines](https://clojurians-log.clojureverse.org/beginners/2021-10-19/1634653649.312300)
```
(clojure.pprint/cl-format nil "lorem lorem ~
lorem lorem ~
lorem lorem")
```

* [Print k, v of hash-map](https://clojurians-log.clojureverse.org/beginners/2022-03-01/1646128570.468909)
```
(doseq [[k v] {:a 5 :b 6 :c 3}]
  (println "Key is" k " and val is" v))
```
