title: Lisp a moderní webové aplikace
author:
  name: Lukáš Rychtecký
  twitter: lukasrychtecky
controls: true
style: style.css

--

# Lisp a moderní webové aplikace
## Lukáš Rychtecký
## [https://twitter.com/LukasRychtecky](https://twitter.com/LukasRychtecky)

--

### Osnova

* Motivace
* ClojureScript: rychlé seznámení
* Reagent: Hello world! Reagent je minimalistický interface k React.js
* Re-frame: Kde bude stav aplikace?
* Komplexní příklad - přihlašovací formulář
* Srovnání Reagent vs Om

--

### Motivace

Single page aplikace:

* Zobrazení dat

```javascript
function renderAvatar(user) {
  var avatarElement = document.createElement('div');
  avatarElement.className = 'avatar';
  avatarElement.appendChild(document.createTextNode(user.fullName));
  return avatarElement;

document.body.appendChild(renderAvatar({fullName: 'Franz Josef'}));
```

--

### Motivace

Single page aplikace:

* Zobrazení dat

```javascript
function renderAvatar(user) {
  var avatarElement = document.createElement('div');
  avatarElement.className = 'avatar';
  avatarElement.appendChild(document.createTextNode(user.fullName));
  return avatarElement;

document.body.appendChild(renderAvatar({fullName: 'Franz Josef'}));
```

* Co když se data změní?

--

### Motivace

Single page aplikace:

* Zobrazení dat

```javascript
function renderAvatar(user) {
  var avatarElement = document.createElement('div');
  avatarElement.className = 'avatar';
  avatarElement.appendChild(document.createTextNode(user.fullName));
  return avatarElement;

document.body.appendChild(renderAvatar({fullName: 'Franz Josef'}));
```

* Co když se data změní? - Prostě funci zavoláme a se UI vyrenderujeme znovu.
* Jak, ale poznat, že se data změnila?

--

### Motivace

#### ReactJS

* Knihovna na zobrazení **stavu** aplikace
* Pure views (pure funkce)
* Renderování stavu do virtuálního DOM
* [https://facebook.github.io/react/](https://facebook.github.io/react/) 

--

### Motivace

* Zobrazení &#x2713;
* Stav

--

### Motivace

* Kde je stav aplikace?
* Jak tento stav měnit?

--

### Motivace

#### Flux

* Framework na správu stavu aplikace
* Async. event driven
* Flux - více stores
* Redux - jeden store
* Lepší výkon s ImmutableJS
* [https://facebook.github.io/flux/](https://facebook.github.io/flux/)
* [http://redux.js.org/](http://redux.js.org/)

--

### Motivace

* Zobrazení &#x2713;
* Stav &#x2713;

--

### Motivace - Současnost

* Clojure (Rich Hickey, 2007, LISP dialect, JVM)
* ClojureScript (Clojure, Javascript machine)

![Clojure creator](img/Rich-Hickey.jpeg)

--

### Motivace - Clojure

* dynamický statický typový
* immutabilní struktury
* pokročilá práce s kolekcemi (vector, linked list, set, hashmap)
* paralelní programování
* lazy sekvence

--

### ClojureScript: rychlé seznámení


* Clojure, který se kompiluje do Javascriptu

```clojure
(+ 1 2) ; 3

(->> (range 9)
     (filter even?)
     (reduce +)) ; 20

(defn add
  [a b]
  (+ a b))

(add 1 2) ; 3
```

--

### ClojureScript + ReactJS = Reagent

* [Reagent](https://reagent-project.github.io/)
* Wrapper nad ReactJS
* Hiccup syntaxe

```clojure
(defn avatar
  [user]
  [:div.avatar (:full-name user)])

(reagent/render [avatar {:full-name "Franz Josef"}] (.-body js/document))
```

--

### Reagent vs. Vanilla Javascript

```clojure
(defn avatar
  [user]
  [:div.avatar (:full-name user)])

(reagent/render [avatar {:full-name "Franz Josef"}] (.-body js/document))
```

vs

```javascript
function renderAvatar(user) {
  var avatarElement = document.createElement('div');
  avatarElement.className = 'avatar';
  avatarElement.appendChild(document.createTextNode(user.fullName));
  return avatarElement;

document.body.appendChild(renderAvatar({fullName: 'Franz Josef'}));
```

--

### Reagent - React life-cycle

```clojure
(defn avatar
  [user]
  (reagent/create-class
    {:display-name
     "avatar"

     :component-did-mount
     (fn []
       ...)

     :reagent-render
     (fn [user]
       [:div
        ...
```

* `create-class` vytvoří React třídu s příslušnými metodami

--

### Re-frame: Kde bude stav aplikace?

* [Re-frame](https://github.com/Day8/re-frame) - "MVC" framework pro UI web. aplikace
* Redux implementovaný čistě v CLJS
* Jeden store
* Změna stavů pomocí eventů

--

### Re-frame: Kde bude stav aplikace?

#### Flow

![Re-frame](https://raw.githubusercontent.com/Day8/re-frame/master/images/logo/re-frame_128w.png)

```text
View -> dispatch -> Handler -> Subscriber
|                                        |
     <-                      <-
```

--

### Re-frame: Kde bude stav aplikace? - view

* definuje UI

Pure view

```clojure
(defn hello-view
  [name]
  [:h1 "Hello " name])
```

Tohle už není pure view

```clojure
(defn hello-view
  []
  (let [name (subscribe [:name])]
    (fn []
      [:h1 "Hello " @name])))
```

--

### Re-frame: Kde bude stav aplikace? - handler

Handler + side effect

* "klasický" handler na události

```clojure
(reg-event-db
  (after write-to-log) ; <-- side effect, zůstává mimo hlavní logiku
  :set-name
  (fn [db [_ name]]
    (assoc db :name name)))
```

--

### Re-frame: Kde bude stav aplikace? - handler s middlewary

Složitější příklad

```clojure
(reg-event-db
  [send-credentials start-progress]
  :login/send-credentials
  (fn [db [_ data]]
    db))
```

* `send-credentials` - volání serveru (side effect je mimo hlavní logiku)
* `start-progress` - změní stav formuláře a zobrazí se třeba spinner (pure function)
* middlewary fungují jako transducery na stavem aplikace

--

### Reagent + Re-frame: subscribers

Subscriber

* pohled na data

```clojure
(reg-sub
  :name
  (fn [db _]
    (get db :name)))
```

--

### Komplexní příklad - přihlašovací formulář

[https://github.com/LukasRychtecky/prague-lambda-cljs-demo](https://github.com/LukasRychtecky/prague-lambda-cljs-demo)

--

###  Srovnání Reagent vs Om

```clojure
(defn main [{:keys [todos showing editing] :as app} comm]
  (dom/section #js {:id "main" :style (hidden (empty? todos))}
    (dom/input
      #js {:id "toggle-all" :type "checkbox"
           :onChange #(toggle-all % app)
           :checked (every? :completed todos)})))
```

* Zdroj [https://github.com/swannodette/todomvc](https://github.com/swannodette/todomvc)
* Defnice views - Hiccup vs. funkce
* Tiny ReactJS wrapper - atributy se předávají s makrem `#js` (camelCase)

--

###  Srovnání Reagent vs Om

```clojure
(when-not (string/blank? (.. new-field -value trim))
        (let [new-todo {:id (guid)
                        :title (.-value new-field)
                        :completed false}]
          (om/transact! app :todos
            #(conj % new-todo)
            [:create new-todo]))
        (set! (.-value new-field) ""))
```

* Modifikace globálního atomu pomocí `update!`, `set!`, `transact!`

```clojure
(defn todo-app [{:keys [todos] :as app} owner]
  (reify
    om/IWillMount
    (will-mount [_]
```

* React life-cycle

--

### Shrnutí

* Reagent - komponenty jsou více clojuristic
* Re-frame poskytuje architekturu aplikace a odstiňuje od modifikace atomu
* Hiccup vs. funkce (osobně preferuji Hiccup)

--

### Odkazy

* [http://www.braveclojure.com/](http://www.braveclojure.com/)
* [https://clojuredocs.org/](https://clojuredocs.org/)
* [https://github.com/clojure/clojurescript](https://github.com/clojure/clojurescript)
* [http://cljs.info/cheatsheet/](http://cljs.info/cheatsheet/)
* [https://github.com/Day8/re-frame](https://github.com/Day8/re-frame)
* [https://reagent-project.github.io/](https://reagent-project.github.io/)
