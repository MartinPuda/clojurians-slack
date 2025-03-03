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

### [Add line numbers](https://clojurians-log.clojureverse.org/clojure/2022-05-10/1652210591.072349)
```
(->> (str/split-lines "foo\nbar\nbaz")
     (map-indexed #(hash-map :line-number %1 :text %2)))

=> ({:line-number 0, :text "foo"} {:line-number 1, :text "bar"} {:line-number 2, :text "baz"})
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

### Remove namespace / unqualify

Task:
```
For example, I have a map like {:myapp/a 1 :myapp/b 2} and I want to get {:a 1 :b 2}
```
My solution ([1](https://clojurians-log.clojureverse.org/beginners/2022-03-07/1646635599.183219),[2](https://clojurians-log.clojureverse.org/beginners/2022-04-07/1649356297.453099)):
```
(-> {:myapp/a 1 :myapp/b 2}
    (update-keys #(keyword (name %))))

(keyword (name :qualified/banana))
=> :banana
```

### [Keys to strings](https://clojurians-log.clojureverse.org/beginners/2022-05-10/1652194274.890839)
```
(update-keys {:a 1 :b 2} name)
=> {"a" 1, "b" 2}
```

### Assoc multiple entries

(upgrade of [multiple calls of assoc](https://clojurians-log.clojureverse.org/beginners/2022-03-07/1646646228.084089))
```
(assoc {} :a 1 :b 2)
```

### [Ratio of filtered to whole](https://clojurians-log.clojureverse.org/beginners/2022-03-11/1647019212.458059)
```
(defn printer-error [s]
  (-> (filter #(> (int %) 109) s)
      count
      (str "/" (count s)))) 
```

### [List of words to list of sentences](https://clojurians-log.clojureverse.org/beginners/2022-03-11/1647022566.519439)
```
(->> (-> (clojure.string/join " " [ "Hello." "How" "are" "you?" "I" "am" "fine."])
         (clojure.string/split #"(?<=\.)|(?<=\?)|(?<=\!)"))
     (mapv clojure.string/trim))
=> ["Hello." "How are you?" "I am fine."]
```

### [Filter seq of strings by seq of substrings](https://clojurians-log.clojureverse.org/beginners/2022-03-27/1648388852.441519)
```
(->> ["who grades foo bar" "who foo bar" "foo bar"]
     (filter #(every? (fn [substring] (str/includes? % substring))
                      ["who" "grade"])))
=> ("who grades foo bar")
```

### [Entries difference](https://clojurians-log.clojureverse.org/beginners/2022-03-28/1648498188.361919)
```
(->> [{:ts "1633392058.124700"} {:ts "1633392055.124600"}]
     (map :ts)
     (map parse-double)
     (apply -))
=> 3.0001001358032227
```

### [Subseqs with atoms](https://clojurians-log.clojureverse.org/beginners/2022-05-07/1651996549.938359)
```
(->> '((1 2 3) ((("a" "b") ([1] ("foo" "bar"))) [10 20]))
     (tree-seq coll? identity)
     (filter #(and (coll? %)
                   (every? (complement coll?) %))))

=> ((1 2 3) ("a" "b") [1] ("foo" "bar") [10 20])
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

### [Date to Epoch Second](https://clojurians-log.clojureverse.org/beginners/2022-03-09/1646830859.419169)

```
(-> "09-03-2022"
    (LocalDate/parse (DateTimeFormatter/ofPattern "dd-MM-yyyy"))
    (.atStartOfDay)
    (.toInstant ZoneOffset/UTC)
    (.getEpochSecond))
```

### [Parse with Offset](https://clojurians-log.clojureverse.org/beginners/2022-04-05/1649166217.759159)
```
(OffsetDateTime/of
  (LocalDateTime/parse
    "2022-03-31T21:35:35Z"
    (DateTimeFormatter/ofPattern "yyyy-MM-dd'T'HH:mm:ssz"))
  (ZoneOffset/of "-06:00"))
```

### [Minus 1 hour](https://clojurians-log.clojureverse.org/clojure/2022-05-02/1651497438.166679)
```
(.minusHours (java.time.LocalDateTime/now) 1)
=> #object[java.time.LocalDateTime 0x46a8689 "2022-05-02T14:16:15.340366900"]

(-> (java.time.LocalDateTime/now)
    (.minusHours 1))
=> #object[java.time.LocalDateTime 0x3c1cec05 "2022-05-02T14:16:37.845970"]

(doto (java.time.LocalDateTime/now)
  (.minusHours 1))
=> #object[java.time.LocalDateTime 0x243b6d46 "2022-05-02T15:18:26.814173600"]

```

### [Format to Pattern](https://clojurians-log.clojureverse.org/beginners/2022-05-07/1651943963.249989)
```
(.format
  (LocalDateTime/parse
    "Wed Aug 19 23:06:10 +0000 2020"
    (DateTimeFormatter/ofPattern "EEE MMM dd HH:mm:ss xxxx yyyy" (Locale. "US")))
  (DateTimeFormatter/ofPattern "yyyy-MM-dd HH-mm-ss" (Locale. "US")))
```
### [To Timestamp](https://clojurians-log.clojureverse.org/beginners/2022-05-07/1652163937.819909)
```
(->> "2020-08-19 23-06-10"
     (.parse (SimpleDateFormat. "yyyy-MM-dd hh-mm-ss"))
     (.getTime)
     (java.sql.Timestamp.))
```


## Strings, output, printing

### [Interleave chars with newline](https://clojurians-log.clojureverse.org/beginners/2021-07-11/1626013784.343500)
```
(clojure.string/join "\n" "This is text")
(clojure.pprint/cl-format false "~{~a~^~%~}" "This is text")
```

### [Interleave strings with newline](https://clojurians-log.clojureverse.org/beginners/2022-04-05/1649152071.068509)
```
(defn svg2pdf [svg-root-element target-box]
  (clojure.string/join "\n"
    ["q"
     "1 0 0 1 29.5 30.5 cm"
     "q"
     "Q"
     "Q"]))
```

### [Escape newlines](https://clojurians-log.clojureverse.org/beginners/2021-10-19/1634653649.312300)
```
(clojure.pprint/cl-format nil "lorem lorem ~
lorem lorem ~
lorem lorem")
```

### [Print k, v of hash-map](https://clojurians-log.clojureverse.org/beginners/2022-03-01/1646128570.468909)
```
(doseq [[k v] {:a 5 :b 6 :c 3}]
  (println "Key is" k " and val is" v))
```

### [Data to table string](https://clojurians-log.clojureverse.org/beginners/2022-04-09/1649535251.312509)
```
(defn print-tsv
  ([ks rows] (->> (map #(str/join "\t" %) rows)
                  (str/join "\n")
                  (str (str/join "\t" ks) "\n")))
  ([rows] (print-tsv (keys (first rows))
                     (map vals rows))))

(print-tsv '({:log "some text", :id 123} {:log "some other text" :id 124}))
=> ":log\t:id\nsome text\t123\nsome other text\t124"
```
Or `(with-out-str (clojure.pprint/print-table [{:log "some text", :id 123} {:log "some other text" :id 124}]))`

## Regex

### [Exclude curly brackets](https://clojurians-log.clojureverse.org/beginners/2022-05-23/1653289128.046739)

```
(->> "we are {{hi.there}} boo§ {{hullo.there.again}}"
     (re-seq #"(?:\{\{)([^\{\}]*)(?:\}\})")
     (map second))

=> ("hi.there" "hullo.there.again")
```

## Clojure Match

### [Usage example](https://clojurians-log.clojureverse.org/clojure/2022-03-27/1648408344.837659)
```
(clojure.core.match/match [:a :b]
  [:a :b] "a b combo"
  [_ :b] "at least b")
```

## Protocols, records

### [Usage example](https://clojurians-log.clojureverse.org/beginners/2022-04-26/1650965820.424429)
```
(defprotocol AbstractReader
  (read-file [this path])
  (read-directory [this path]))

(defrecord Reader [exts ctx-fn]
  AbstractReader
  (read-file [{:keys [exts ctx-fn]} path]
    (prn exts ctx-fn path))
  (read-directory [{:keys [exts ctx-fn]} path]
    (prn exts ctx-fn path)))

(let [my-reader (->Reader 5 5)]
  (read-file my-reader "path")
  (read-directory my-reader "path"))
```

### Eval Reader Macro

[Example for read-string](https://clojurians-log.clojureverse.org/beginners/2022-05-12/1652350178.750199)

## Code reviews

### [Path to ns](https://clojurians-log.clojureverse.org/clojure/2022-06-04/1654348357.211349)
```
(defn path-to-ns [path]
    (->> (-> (str/replace path #"-" "_")
             (str/split #"/")
             (conj "main")
             rest)
         (str/join ".")))
```
