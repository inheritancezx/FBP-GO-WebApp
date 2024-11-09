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
![alt text](/img/image.png)

