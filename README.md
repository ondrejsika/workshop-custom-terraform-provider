# Workshop: Custom Terraform Provider

    2020 Ondrej Sika <ondrej@ondrejsika.com>
    https://github.com/ondrejsika/workshop-custom-terraform-provider

Alias: [sika.link/if20-terraform](https://sika.link/if20-terraform)


## Related Repositories

- Demo Cloud Provider (Api Server) - https://github.com/ondrejsika/demo-cloud-provider
- Terraform Provider for Demo Cloud (our target) - https://github.com/ondrejsika/terraform-provider-demo

## See API of our Demo Cloud

API Server - https://github.com/ondrejsika/demo-cloud-provider/blob/master/server.py

### API

#### GET /v1/box/

Response:

```json
[
  {"id": 1, "name": "Box 1"},
  {"id": 2, "name": "Box 2"}
]
```

#### POST /v1/box/

Request:

```json
{"name": "Box 1"}
```

Response:

```json
{"box_id": 1}
```

#### GET /v1/box/:box_id/

Response:

```json
{"id": 1, "name": "Box 1"}
```

#### PUT /v1/box/:box_id/

Request:

```json
{"name": "Box 1 Update"}
```

Response:

```json
{"ok": true}
```

#### DELETE /v1/box/:box_id/

Response:

```json
{"ok": true}
```

#### GET /v1/box/:box_id/text/

Response:

```json
[
  {"id": 1, "box_id": 1, "text": "Text 1"},
  {"id": 2, "box_id": 1, "text": "Text 2"}
]
```

#### POST /v1/box/:box_id/text/

Request:

```json
{"text": "Text 1"}
```

Response:

```json
{"text_id": 1}
```

#### GET /v1/box/:box_id/text/:text_id/

Response:

```json
{"id": 1, "box_id": 1, "text": "Text 1"}
```

#### PUT /v1/box/:box_id/text/:text_id/

Request:

```json
{"text": "Text 1 Update"}
```

Response:

```json
{"ok": true}
```

#### DELETE /v1/box/:box_id/text/:text_id/

Response:

```json
{"ok": true}
```


## Setup Go Project for terraform-provider-demo

Create a project directory

```
cd ~/go
mkdir -p src/github.com/ondrejsika/terraform-provider-demo
cd src/github.com/ondrejsika/terraform-provider-demo
```

Init Go module

```
go mod init
```

See `go.mod`:

```
cat go.mod
```

## Create API Helpers

Download helpers:

```
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/api.go
```

We use Resty library, we have to install it:

```
go get -v github.com/go-resty/resty/v2
```

See `go.mod` and `go.sum`:

```
cat go.mod
cat go.sum
```

Create an example `main.go`:

```go
// main.go
package main

import "fmt"

func main(){
	res, _ := GetApi("https://demo-cloud-dev.panda.k8s.oxs.cz/", "xxx", "/v1/box/")
	fmt.Println(res)
}
```

Try our API Helpers:

```
go build && ./terraform-provider-demo
```

## Create API for our Demo Cloud

```
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/box_api.go
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/text_api.go
```

Create an example `main.go`:

```go
// main.go
package main

import "fmt"

func main(){
	res, _ := ListBoxesApi("https://demo-cloud-dev.panda.k8s.oxs.cz/", "xxx")
	fmt.Println(res)
}
```

Try our API:

```
go build && ./terraform-provider-demo
```


## Create Provider for our Demo Cloud

Download a Terraform Provider files:

```
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/resource_text.go
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/resource_box.go
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/provider.go
rm main.go
wget https://raw.githubusercontent.com/ondrejsika/terraform-provider-demo/master/main.go
```


## Create Terraform Manifest

```terraform
//terraform.tf
provider "demo" {
  api_origin = "https://demo-cloud-prod.panda.k8s.oxs.cz"
  token = "xxx"
}

resource "demo_box" "primary" {
  name = "Primary Box"
}

resource "demo_text" "hello" {
  box_id = demo_box.primary.id
  text = "This is my first box"
}

resource "demo_text" "foo" {
  box_id = demo_box.primary.id
  text = "Foo bar ..."
}

resource "demo_box" "secondary" {
  name = "Secondary Box"
}

resource "demo_text" "secondary" {
  box_id = demo_box.secondary.id
  text = "..."
}
```

Build, init, and apply Terraform configuration

```
go build && terraform && terraform apply
```
