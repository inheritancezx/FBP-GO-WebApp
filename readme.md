<div align=center>

# Framework Base Programming <br> Web-App Golang Assignment
Fayza Aqila Bachtiar - 5025221087

</div>

## data structures
A wiki consists of a series of interconnected pages, each of which has a title and a body (the page content). This will display a sample page within the terminal

```go
type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return os.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := os.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}
```

By importing the std go library to the program and having it declares by a struct `page`, a method save `loadpage`, and a function `loadpage`. Main functions shall look as so:

```go
func main() {
	p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
	p1.save()
	p2, _ := loadPage("TestPage")
	fmt.Println(string(p2.Body))
}
```

to run the page we use commands:
```
go build wiki.go
./wiki
```
the display will look like so
![alt text](/img/image1.png)

## net/http 
### interlude
previously we have display the sample page, now we are able to apply it to the network page, using personal network route  

```go
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
adding a `handling` function to the file as well as `log` and `net/http` go library. to enable it we use the `apache tomcat` webserver to display the page. enter `http://localhost:8080/monkeys` to a browser.

![alt text](/img/image2.png)

### wiki app
as now we can add a new route and display by adding a new `handler` func as well as storing the data to a `test.txt`, u might as well simply add `hello world` in the .txt

changes in `wiki.go`
```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	p, _ := loadPage(title)
	fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}

func main() {
	... 
	http.HandleFunc("/view/", viewHandler)
    ...
}
```

the display shall be:
![alt text](/img/image3.png)

### edit/save 
a wiki page wont be perfect without any functionality to edit and save. to add these features are adding up several new functions, a render display, as well as the template of the view using `html`

### functions
```go
func editHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/edit/"):]
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/save/"):]
	body := r.FormValue("body")
	p := &Page{Title: title, Body: []byte(body)}
	p.save()
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

...

func main() {
    ...
	http.HandleFunc("/edit/", editHandler)
	http.HandleFunc("/save/", saveHandler)
    ...
}
```

we will need new functions `edit` and `save` handler to make the functionality itself. then adding the page call in the main function. 

### render func
to enable the ability of the system to display the edit and save, we will need a render to display the `html` template, there is where a render func comes in handy

```go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	t, _ := template.ParseFiles(tmpl + ".html")
	t.Execute(w, p)
}
```

### html
and lastly will be the html files itself. in this webApp, there will be 2 `.html` files, namely `edit.html` and `view.html`. both of them are templates for each of their purposes

- `edit.html`
```html
<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
    <div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
    <div><input type="submit" value="Save"></div>
</form>
```

- `view.html`
```html
<h1>{{.Title}}</h1>

<p>[<a href="/edit/{{.Title}}">edit</a>]</p>
<div>{{printf "%s" .Body}}</div>
```

the display shall look like this:
<p align="center">
  <img src="/img/image4.png" alt="Image 4" width="45%">
  <img src="/img/image5.png" alt="Image 5" width="45%">
</p>

### 404 not found
when we hit a page that doesn't exist, it shall display a not found page landing, this will need changes in the `view` handler 
```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}
```

and this will look like so:
![alt text](/img/image6.png)

## error handling and validation
to handle errors in the prgram, we will add these lines of code to the `save` handler and `render` func. However, the render func shall face slight differences with additional template caching method (to simplify the code structures)

- `saveHandler`
```go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

- `render`
```go
var templates = template.Must(template.ParseFiles("edit.html", "view.html"))

...
...

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    err := templates.ExecuteTemplate(w, tmpl+".html", p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```
first declare a variable namely `templates` then arrange the `.html` files as the template accordingly to later on be a variable call in the function

As for validaton, its slight differences will be confirming wether a title of the page is correct or not. To do so, we will need a new `getTitle` func and calling that func within ot handlers.

```go
func getTitle(w http.ResponseWriter, r *http.Request) (string, error) {
    m := validPath.FindStringSubmatch(r.URL.Path)
    if m == nil {
        http.NotFound(w, r)
        return "", errors.New("invalid Page Title")
    }
    return m[2], nil // The title is the second subexpression.
}
```

to call the func, will be directing them directly to an error management branch:
- `view`
```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
```

- `edit`
```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
```

- `save`
```go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err = p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

## func literals
to accomodate efficiency in the program, we will utilize what is said as `function literals` where it eases the process of error handling as well as validation in within the code. For it we add a func namely `makeHandler`

```go
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        m := validPath.FindStringSubmatch(r.URL.Path)
        if m == nil {
            http.NotFound(w, r)
            return
        }
        fn(w, r, m[2])
    }
}
```
The closure returned by makeHandler is a function that takes an `http.ResponseWriter` and `http.Request` then extracts the `title` from the request path, and validates it with the `validPath regexp`. then The hanlders will also change accordingly

```go
func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
	body := r.FormValue("body")
	p := &Page{Title: title, Body: []byte(body)}
	err := p.save()
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

func main() {
	http.HandleFunc("/", handler)
	http.HandleFunc("/view/", makeHandler(viewHandler))
	http.HandleFunc("/edit/", makeHandler(editHandler))
	http.HandleFunc("/save/", makeHandler(saveHandler))
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This then shall be the final look of the `handler`(s) and the call in the main func.
thank you!  𓆝⋆｡˚ 𓇼