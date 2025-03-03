# clojurians-slack

Some of my contributions to [Clojurians Slack](http://clojurians.net/).

Each solution is also available at [Clojurians Slack Archive](https://clojurians-log.clojureverse.org/).

## Data Transformations

### Nest sequence

Example:
```
input: (1 2 :nest 3 4 :nest 5 6 7 :nest 8)

output: (1 2 (3 4 (5 6 7 (8))))
```

My solution ([1](https://clojurians-log.clojureverse.org/clojure/2023-01-24/1674543932.399009),[2](https://clojurians-log.clojureverse.org/clojure/2023-01-24/1674547251.794549)):
```
(defn nest-seq [coll]
  (when (seq coll)
    (let [[f & r] coll]
      (if (= f :nest)
        (list (nest-seq r))
        (cons f (nest-seq r))))))

(nest-seq '(1 2 :nest 3 4 :nest 5 6 7 :nest 8))
=> (1 2 (3 4 (5 6 7 (8))))

; For nested input seq

(defn nest-seq [coll]
  (if (empty? coll) '()
    (let [[f & r] coll]
      (cond
        (sequential? f) (cons (nest-seq f)
                              (nest-seq r))
        (= f :nest) (list (nest-seq r))
        :else (cons f (nest-seq r))))))

```

### Transform vector

Example:
```
I have a vector of timestamp [1600 1601 1602 1603] and another matrix containing some value [ [1600 1] [1603 3]] . I want final result like [[1600 1] "N/A" "NA " [1603 3]].
```

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
```
([1 0] [2 1] [3 0] [4 1] [5 0] [6 1]) to  ((1 3 5) (2 4 6))
```

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

Example:
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

### [Sort by value](https://clojurians-log.clojureverse.org/beginners/2022-06-14/1655208273.278699)
```
(->> [{:Number 1, :value 11} {:Number 2, :value 7} {:Number 3, :value 78}]
     (sort-by :value >))

=> ({:Number 3, :value 78} {:Number 1, :value 11} {:Number 2, :value 7})
```
### [Partition when truthy](https://clojurians-log.clojureverse.org/beginners/2022-06-25/1656221936.551449)
```
(->> [1 3 2 4 1 2 3]
     (partition-by (let [v (atom 0)]
                     #(if (odd? %) 
                        (swap! v inc) 
                        @v))))

=> ((1) (3 2 4) (1 2) (3))
```
### [.contains](https://clojurians-log.clojureverse.org/beginners/2022-07-29/1659087575.905159)
```
(.contains [[:a :b] [:c :d] [:e :f]] [:a :b])
=> true
```

### [Item to category](https://clojurians-log.clojureverse.org/beginners/2022-08-01/1659359531.165349)
```
(defn to-category [n]
  (->> [1 101 121 151]
       (take-while #(>= n %))
       count))
```
### [Partition by even index](https://clojurians-log.clojureverse.org/beginners/2022-08-02/1659425334.774789)
```
(let [xs [0 1 2 3 4 5]]
  [(take-nth 2 xs)
   (take-nth 2 (rest xs))])

=> [(0 2 4) (1 3 5)]
```
### [Partition to columns](https://clojurians-log.clojureverse.org/beginners/2022-08-04/1659605112.075339)
```
(->> [1 2 3 4 5 6 7 8 9 10 11 12 13 14 15  16]
     (partition 4)
     (apply map vector))

=> ([1 5 9 13] [2 6 10 14] [3 7 11 15] [4 8 12 16])
```
### Split on values
Example:
```
(split nil nil) -> nil
(split '(1 2 3) '(true false)) -> ((1) (2 3))
(split '(1 2 3) '(false true)) -> ((1 2) (3))
(split '(1 2 3) '(true true)) -> ((1) (2) (3))
```
[My solution](https://clojurians-log.clojureverse.org/beginners/2022-09-16/1663363687.771219):
```
(defn split [v1 v2]
  (->> (interleave v1 (conj v2 (peek v2)))
       (partition-by true?)
       (remove #{[true]})
       (map #(remove false? %))))

(split nil nil)
=> ()
(split '(1 2 3) [true true])
=> ((1) (2) (3))
(split '(1 2 3) [false true])
=> ((1 2) (3))
(split '(1 2 3) [true false])
=> ((1) (2 3))
```
### Data to hash-map
Example:
```
({:name "name1", :id "61fd12debd75d9413d874a42"}
 {:name "name2", :id "61dd3720ac8ceb2d1f4f8419"}
 {:name "name3", :id "61dd404da88a335e39ac984b"})
;into 
"61fd12debd75d9413d874a42" name1
"61dd3720ac8ceb2d1f4f8419" name2
...
```
[My solution](https://clojurians-log.clojureverse.org/beginners/2022-09-28/1664370060.704369) (`reduce` variants: [1](https://clojurians-log.clojureverse.org/beginners/2022-09-28/1664370581.030139),[2](https://clojurians-log.clojureverse.org/beginners/2022-09-28/1664371426.920899),[3](https://clojurians-log.clojureverse.org/beginners/2022-09-28/1664371762.858699)):
```
(->> '({:name "name1", :id "61fd12debd75d9413d874a42"}
       {:name "name2", :id "61dd3720ac8ceb2d1f4f8419"}
       {:name "name3", :id "61dd404da88a335e39ac984b"})
     (mapv (juxt :id :name))
     (into {}))
```

### [Set difference](https://clojurians-log.clojureverse.org/beginners/2022-08-05/1659700884.241839)
```
(clojure.set/difference (set {:a 2 :c 2 :d 3})
                        (set {:a 1 :c 2 :d 3}))
                        
=> #{[:a 2]}
```
### [Max in each group](https://clojurians-log.clojureverse.org/beginners/2022-08-23/1661238386.521359)
```
(->> (group-by :version d)
     (map (fn [[_ rows]] (last (sort-by :date rows)))))

(->> (group-by :version d)
     vals
     (map #(last (sort-by :date %))))
```
### [Count occurences of kv pair](https://clojurians-log.clojureverse.org/beginners/2022-11-19/1669314582.394249)
```
(defn count-pair
  [data k v]
  (->> data
       (tree-seq #(or (map? %) (vector? %)) seq)
       (filter #(and (map-entry? %) (= % [k v])))
       count))
```
### [Group hash-map keys by namespace](https://clojurians-log.clojureverse.org/beginners/2022-11-24/1669302148.531609)
```
(-> (group-by (fn [[k v]] (namespace k)) {:a/x 1 :a/y 2 :b/x 1 :b/y 2})
    (update-keys keyword)
    (update-vals #(into {} %)))
=> {:a #:a{:x 1, :y 2}, :b #:b{:x 1, :y 2}}
```
### Combinations([1](https://clojurians-log.clojureverse.org/beginners/2023-02-07/1675765800.037539),[2](https://clojurians-log.clojureverse.org/beginners/2023-02-07/1675766719.167739))
```
(clojure.math.combinatorics/combinations '(10 20 11 30) 2)
=> ((10 20) (10 11) (10 30) (20 11) (20 30) (11 30))

; in specific order
(defn comb2 [coll]
  (for [x (range 1 (count coll))
        y (range 0 x)]
    [(nth coll y) (nth coll x)]))

(comb2 [10 20 11 30])
=> ([10 20] [10 11] [20 11] [10 30] [20 30] [11 30])
(comb2 '(10 20 11 30))
=> ([10 20] [10 11] [20 11] [10 30] [20 30] [11 30])
```
### Cartesian product over map
Example:
```
Say I’ve got a map of keys and sequential values like:

{:k [k1 k2 k3] :l [l1 l2]}
How would I go about creating the permutations of all the keys and values in separate maps? e.g something like:

[{:k k1 :l l1} {:k k2 :l :l1} … {:k k3 :l l2}]
```

[My solution](https://clojurians-log.clojureverse.org/clojure/2023-03-01/1677689447.470619):
```
(defn perm [m]
  (let [ks (keys m)
        vs (vals m)]
    (mapv #(zipmap ks %)
          (apply combi/cartesian-product vs))))

(perm '{:k [k1 k2 k3] :l [l1 l2]})
=> [{:k k1, :l l1} {:k k1, :l l2} {:k k2, :l l1} {:k k2, :l l2} {:k k3, :l l1} {:k k3, :l l2}]
```

### [Index of sublist](https://clojurians-log.clojureverse.org/beginners/2023-03-03/1677837236.340799)
```
(java.util.Collections/indexOfSubList [0 13 8 100 20 8 9 14] [100 20 8])
=> 3
```

## Juxt

### [Juxt usage](https://clojurians-log.clojureverse.org/beginners/2021-12-02/1638479674.293500)
```
(let [[x y] ((juxt min max) 1 2 3 4)]
  (println x y))
1 4
=> nil
```
### [Juxt split](https://clojurians-log.clojureverse.org/clojure/2022-12-15/1671080341.147289)
```
; traverses twice
((juxt filter remove) odd? (range 10))
=> [(1 3 5 7 9) (0 2 4 6 8)]

; this traverses once
(let [result (group-by pred coll)]
  [(get result true) (get result false)])
```
### [Juxt sort](https://clojurians-log.clojureverse.org/beginners/2023-02-19/1676832821.868129)
Sort by asc + desc.
```
(->> data
     (sort-by (juxt #(- (:score %)) :size)))
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
### [Parse with Locale](https://clojurians-log.clojureverse.org/beginners/2023-01-30/1675101018.717449)
```
(LocalDate/parse "Jan 30 2023" (DateTimeFormatter/ofPattern "MMM dd yyyy"
                                                            Locale/UK))
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
### [Format to AM/PM](https://clojurians-log.clojureverse.org/clojure/2023-02-20/1676907438.845199)
```
(.format (DateTimeFormatter/ofPattern "hh:mm a" Locale/US)
         (LocalDateTime/ofInstant (Instant/ofEpochSecond 1676901600)
                                  ZoneOffset/UTC))
=> "02:00 PM"

(.format (DateTimeFormatter/ofPattern "h:mm a" Locale/US)
         (LocalDateTime/ofInstant (Instant/ofEpochSecond 1676901600)
                                  ZoneOffset/UTC))
=> "2:00 PM"
```
### [To Timestamp](https://clojurians-log.clojureverse.org/beginners/2022-05-07/1652163937.819909)
```
(->> "2020-08-19 23-06-10"
     (.parse (SimpleDateFormat. "yyyy-MM-dd hh-mm-ss"))
     (.getTime)
     (java.sql.Timestamp.))
```
### [Current DT for Zone](https://clojurians-log.clojureverse.org/beginners/2022-06-18/1655538066.121149)
```
(.format (DateTimeFormatter/ofPattern "yyyy-MM-dd HH:mm:ss")
         (ZonedDateTime/now (ZoneId/of "Europe/Berlin")))

=> "2022-06-18 09:40:31"
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

### [Pad from right](https://clojurians-log.clojureverse.org/beginners/2022-06-08/1654685232.009739)
```
; org.apache.commons.lang3.StringUtils
(StringUtils/rightPad "Breaded Mushrooms" 25 \.)
=> "Breaded Mushrooms........"
```
### [Strip char](https://clojurians-log.clojureverse.org/beginners/2022-09-01/1662039856.491529)
```
; org.apache.commons.lang3.StringUtils
(StringUtils/strip "a foo a" "a")
=> " foo "
```
Or `str/replace`

### [Parse with Locale](https://clojurians-log.clojureverse.org/beginners/2022-06-20/1655744349.637029)
```
(.parse (NumberFormat/getInstance Locale/US)
        "1,505,000")

=> 1505000
```
### [RuleBasedNumberFormat](https://clojurians-log.clojureverse.org/clojure/2022-08-26/1661528535.253329)
```
(let [nf (RuleBasedNumberFormat. Locale/UK RuleBasedNumberFormat/ORDINAL)]
  (dotimes [i 10]
    (println i (.format nf i "%digits-ordinal"))))

(let [nf (RuleBasedNumberFormat. Locale/FRENCH RuleBasedNumberFormat/ORDINAL)]
  (dotimes [i 10]
    (println i (.format  nf i "%digits-ordinal"))))
```

## Regex

### [Exclude curly brackets](https://clojurians-log.clojureverse.org/beginners/2022-05-23/1653289128.046739)

```
(->> "we are {{hi.there}} boo§ {{hullo.there.again}}"
     (re-seq #"(?:\{\{)([^\{\}]*)(?:\}\})")
     (map second))

=> ("hi.there" "hullo.there.again")
```

## Filesystem

### [Read subvec of chars](https://clojurians-log.clojureverse.org/beginners/2022-08-07/1659884254.481899)
```
(subvec (-> (slurp "...")
            (s/split-lines))
        start end)
```
### [Traverse, print number of files](https://clojurians-log.clojureverse.org/beginners/2022-09-06/1662467300.679699)
```
(defn traverse [file]
  (when (.isDirectory file)
    (let [files (seq (.listFiles file))]
      (println (.getAbsolutePath file) ":" (count files))
      (run! traverse files))))

(traverse (File. "."))
```
### [Traverse, merge jsons](https://clojurians-log.clojureverse.org/beginners/2022-10-31/1667207371.709499)
Add `spit` to some file:
```
(defn merge-jsons
  "file should be java.io.File, eg (File. string-path)"
  [file]
  (let [all-files (.listFiles file)
        subfolders (filter #(.isDirectory %) all-files)
        subfiles (filter #(.isFile %) all-files)]
    (->> subfiles
         (map #(json/read-value (slurp %)))
         (json/write-value-as-bytes))
    (doseq [f subfolders]
      (merge-jsons f))))

; file-seq and .listFiles have different results (`file-seq` returns all files, .listFiles only these on the current "level").
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

## Eval Reader Macro

[Example for read-string](https://clojurians-log.clojureverse.org/beginners/2022-05-12/1652350178.750199)

## Meta

### [Copy Meta](https://clojurians-log.clojureverse.org/clojure/2022-07-31/1659240765.955429)
```
(-> (def var1 value)
    (reset-meta! (meta #'var2)))
```
## [Type Hints](https://clojurians-log.clojureverse.org/clojure/2022-12-16/1671223215.450749)
```
(class (into-array ["a" "b"]))
=> [Ljava.lang.String;

; Type hint will be: ^"[Ljava.lang.String;"
```
### [New type hints for Java methods](https://clojurians-log.clojureverse.org/beginners/2024-02-17/1708178694.409279)
```
(apply str (filter ^[char] Character/isUpperCase "FooBarBaz"))
=> "FBB"
```
## Ns

### [Public vars](https://clojurians-log.clojureverse.org/beginners/2023-02-02/1675337518.333069)
```
(vals (ns-publics 'clojure.core))
```
### [Only functions](https://clojurians-log.clojureverse.org/beginners/2023-02-02/1675340181.771829)
```
(filter #(and  (:arglists (meta %))
               (not (:macro (meta %))))
        (vals (ns-publics 'clojure.core)))
```

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
### [Resize Image](https://clojurians-log.clojureverse.org/beginners/2022-06-23/1656004297.263769)
```
(defn resize
  "Resize a file.  Pass in a width and height."
  [file-in file-out width height]
  (try (let [img (ImageIO/read (io/file file-in))
             simg (BufferedImage. width height BufferedImage/TYPE_INT_RGB)
             g (.getGraphics simg)
             scaled (.getScaledInstance img width height Image/SCALE_SMOOTH)]
         (.drawImage g scaled 0 0 width height nil)
         (ImageIO/write simg "jpeg" (io/file file-out)))
       (catch Exception e (prn e))))
```
### Count increasing pairs ([1](https://clojurians-log.clojureverse.org/beginners/2022-08-21/1661106488.197009),[2](https://clojurians-log.clojureverse.org/beginners/2022-08-21/1661107414.334109))
```
(defn count-inc [coll]
  (->> (partition 2 1 coll)
       (filter (fn [[a b]]
                 (> b a)))
       count))

(defn count-inc [coll]
  (reduce (fn [n [a b]]
            (if (> b a) (inc n) n))
          0 
          (partition 2 1 coll)))

(count-inc [0 2 4 6])
=> 3
```
### Search in tree-seq ([1](https://clojurians-log.clojureverse.org/beginners/2023-01-27/1674849260.340859),[2](https://clojurians-log.clojureverse.org/beginners/2023-01-27/1674850147.738329),[3](https://clojurians-log.clojureverse.org/beginners/2023-01-27/1675288890.462689))
```
; 1
(def data
  [{:name "a" :children [{:name "b" :children [{:name "c" :children []}]}
                         {:name "d" :children []}]}])

(defn in-tree? [data item]
  (.contains (tree-seq associative? identity data)
             [:name item]))

(defn search-name [{:keys [name children] :as data} searched-name]
  (if (= name searched-name)
    (assoc data :children [])
    (assoc data :children [(->> children
                                (filter #(in-tree? % searched-name))
                                (#(search-name (first %) searched-name)))])))

(defn search [data searched-name]
  (when (in-tree? data searched-name)
    (search-name data searched-name)))

(search (first data) "a")
=> {:name "a", :children []}
(search (first data) "b")
=> {:name "a", :children [{:name "b", :children []}]}
(search (first data) "c")
=> {:name "a", :children [{:name "b", :children [{:name "c", :children []}]}]}
(search (first data) "d")
=> {:name "a", :children [{:name "d", :children []}]}
(search (first data) "e")
=> nil

; 2
(defn in-tree? [data item]
  (.contains (tree-seq associative? identity data)
             [:name item]))

(defn search [{:keys [name children] :as data} searched-name]
  (when (in-tree? data searched-name)
    (if (= name searched-name)
      (assoc data :children [])
      (assoc data :children [(into {} (mapcat #(search % searched-name) children))]))))

; 3
(defn includes-in-tags-tree? [tags substr]
  (->> (tree-seq associative? identity tags)
       (some #(and (map-entry? %)
                   (let [[k v] %]
                     (and (= k :name)
                          (str/includes? v substr)))))))

(includes-in-tags-tree? (first data) "a")                   ;; => true
(includes-in-tags-tree? (first data) "b")                   ;; => true
(includes-in-tags-tree? (first data) "c")                   ;; => true
(includes-in-tags-tree? (first data) "d")                   ;; => true
```
### Fold table
Example:
```
Hello All, I have data that looks like this:

| :FieldName | :DimensionScheme | :SortRank |
|------------|------------------|----------:|
|     FAMILY |       FAMILY-SKU |         1 |
|     SKU_ID |       FAMILY-SKU |         2 |
This is in a http://tech.ml dataset format. I also can convert it to a map, which would look like this:
[{:FieldName "FAMILY", :DimensionScheme "FAMILY-SKU", :SortRank 1} 
 {:FieldName "SKU_ID", :DimensionScheme "FAMILY-SKU", :SortRank 2}]
Either way, I am trying to write a function that gives me the following output: {:DimensionScheme "FAMILY-SKU", :group-by-params ["FAMILY" "SKU_ID"]} With the following constraints: 1. If there are multiple DimensionSchemes in the map, I would like to have a row/map entry for each DimensionScheme. 2. The position of the :FieldName string within the group-by-params vector has to match the :SortRank in the input data.
```
[My solution](https://clojurians-log.clojureverse.org/beginners/2023-02-02/1675365283.857439):
```
(def data [{:FieldName "FAMILY", :DimensionScheme "FAMILY-SKU", :SortRank 1}
           {:FieldName "SKU_ID", :DimensionScheme "FAMILY-SKU", :SortRank 2}])

(->> data
     (group-by :DimensionScheme)
     (mapv (fn [[k v]] {:DimensionScheme k
                        :group-by-params (->> v (sort-by :SortRank) (mapv :FieldName))})))

;=> [{:DimensionScheme "FAMILY-SKU", :group-by-params ["FAMILY" "SKU_ID"]}]
```
### Tree from data
Example:
```
({:context ["root"], :values "lalala1"}
 {:context ("root" "select-location"), :values "lalala2"}
 {:context ("root" "select-location"), :values "lalala3"}
 {:context ("root" "select-vendor"), :values "lalala4"}
 {:context ("root" "select-vendor"), :values "lalala5"}
 {:context ("root" "select-vendor"), :values "lalala6"}
 {:context ["root"], :values "lalala7"}
 {:context ["root"], :values "lalala8"})

{:context "root", 
 :values ["lalala1"
          {:context "select-location" 
           :values ["lalala2"
                    "lalala3"]}
          {:context "select-vendor"
           :values "lalala4"
                   "lalala5"
                   "lalala6"}
          "lalala7"
          "lalala8"]}
```
[My solution](https://clojurians-log.clojureverse.org/clojure/2024-03-02/1709378967.435739):
```
(defn solution [data]
  (into [] (comp (partition-by (comp first :context))
                 (mapcat (fn [part]
                           (if (empty? (-> part first :context))
                             (mapv :values part)
                             [{:context (-> part first :context first)
                               :values  (solution (mapv #(update % :context rest) part))}]))))
        data))

(solution data)
=>
[{:context "root",
  :values ["lalala1"
           {:context "select-location", :values ["lalala2" "lalala3"]}
           {:context "select-vendor", :values ["lalala4" "lalala5" "lalala6"]}
           "lalala7"
           "lalala8"]}]
```
