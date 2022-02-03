## Release

`bb release-artifact v0.0.1`

## Example

``` clojure
(ns garden-watch-task
  (:require [babashka.pods :as pods]
            [babashka.fs :as fs]
            [clojure.java.io :as io]
            [clojure.edn :as edn]))

(pods/load-pod "pods/pod-babashka-garden")
(require '[pod.babashka.garden :as garden])

(pods/load-pod 'org.babashka/fswatcher "0.0.2")
(require '[pod.babashka.fswatcher :as fw])

(defn read-all [s]
  (with-open [reader (io/reader s)]
    (let [reader' (java.io.PushbackReader. reader)
          f (fn f []
                (let [v (edn/read {:eof ::end} reader')]
                  (when-not (= ::end v)
                    (cons v (f)))))]
      (f))))

(def input "src/main/custom.css.clj")
(def output "out/web/public/css/custom.css")

(defn generate []
  (prn "Regenerate" output)
  (->> (io/file input)
       (read-all)
       (apply garden/css)
       (spit output)))

(defn watch []
  (fs/create-dirs "out/web/public/css/")
  (generate)
  (fw/watch input
            (fn [e]
              (case (:type e)
                (:write|chmod :write) (generate)
                nil))
            {:delay-ms 100})
  (deref (promise)))

```

