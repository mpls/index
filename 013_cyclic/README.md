<!-- __JSON: gobin -m -run myitcv.io/cmd/egrunner script.sh # LONG ONLINE

## Cyclic module dependencies

Cyclic module dependencies typically come about through test dependencies. This example presents a simple cyclic
dependency between two packages, through one's external test package. Those two packages are initially part of the same
module, but then split into two separate modules where the cyclic dependency is created.

The resulting repository state can be seen at {{PrintOut "repo"}}.

### Walk-through

Create a repository root and add a git remote:

```
{{PrintBlock "setup" -}}
```

Define a module at the root of the repo:

```
{{PrintBlock "define repo root module" -}}
```

Add two subpackages `a` and `b`, with a dependency from `a -> b` and `b_test -> a`:

```go
{{PrintBlock "cat a" -}}
```

```go
{{PrintBlock "cat b" -}}
```

```go
{{PrintBlock "cat b_test" -}}
```

Our directory structure now looks like this:

```
{{PrintBlockOut "tree" -}}
```

Test:

```
{{PrintBlock "test" -}}
```

Commit and push:

```
{{PrintBlock "commit and push" -}}
```

Create a submodule from the subpackage `b`. Note at this point, only `{{PrintBlockOut "module"}}` can be resolved over
the internet; `{{PrintBlockOut "moduleb"}}` only exists locally. This will also be the case for any CI run before our
next commit is referenced by `master` on our remote. Hence we temporarily use relative `replace` directives for the
dependencies `a -> b` and `b -> a`. However, because of [#26241](https://github.com/golang/go/issues/26241), we cannot
currently use a `replace` directive to satisfy the dependency `a -> b`. So for now this next step is pushed in a "broken"
state; we don't add relative `replace` directives for either dependency. This step will be updated once
[#26241](https://github.com/golang/go/issues/26241) is fixed.

```
{{PrintBlock "create submodule from b" -}}
```

Commit and push:

```
{{PrintBlock "commit and push b submodule" -}}
```

Until [#26241](https://github.com/golang/go/issues/26241) is merged, this is where the mutual dependency gets created:

```
{{PrintBlock "create mutual dependency" -}}
```

List the dependencies of `{{PrintBlockOut "module"}}`:

```
{{PrintBlock "list root dependencies" -}}
```

List the dependencies of `{{PrintBlockOut "moduleb"}}`:

```
{{PrintBlock "list b dependencies" -}}
```

Commit the mutual dependency:

```
{{PrintBlock "commit mutual dependency" -}}
```

### Version details

```
{{PrintBlockOut "version details" -}}
```

-->

## Cyclic module dependencies

Cyclic module dependencies typically come about through test dependencies. This example presents a simple cyclic
dependency between two packages, through one's external test package. Those two packages are initially part of the same
module, but then split into two separate modules where the cyclic dependency is created.

The resulting repository state can be seen at https://github.com/go-modules-by-example/cyclic.

### Walk-through

Create a repository root and add a git remote:

```
$ mkdir -p /home/gopher/scratchpad/cyclic
$ cd /home/gopher/scratchpad/cyclic
$ git init -q
$ git remote add origin https://github.com/go-modules-by-example/cyclic
```

Define a module at the root of the repo:

```
$ go mod init
go: creating new go.mod: module github.com/go-modules-by-example/cyclic
```

Add two subpackages `a` and `b`, with a dependency from `a -> b` and `b_test -> a`:

```go
$ catfile a/a.go
$ cat a/a.go
package a

import "github.com/go-modules-by-example/cyclic/b"

const AName = b.BName
```

```go
$ catfile b/b.go
$ cat b/b.go
package b

const BName = "B"
```

```go
$ catfile b/b_test.go
$ cat b/b_test.go
package b_test

import (
	"fmt"
	"github.com/go-modules-by-example/cyclic/a"
	"testing"
)

func TestUsingA(t *testing.T) {
	fmt.Printf("Here is A: %v\n", a.AName)
}
```

Our directory structure now looks like this:

```
.
|-- a
|   `-- a.go
|-- b
|   |-- b.go
|   `-- b_test.go
`-- go.mod
```

Test:

```
$ go test -v ./...
?   	github.com/go-modules-by-example/cyclic/a	[no test files]
=== RUN   TestUsingA
Here is A: B
--- PASS: TestUsingA (0.00s)
PASS
ok  	github.com/go-modules-by-example/cyclic/b	0.003s
```

Commit and push:

```
$ git add -A
$ git commit -q -am "Commit 1: initial commit of parent module github.com/go-modules-by-example/cyclic"
$ git rev-parse HEAD
7463c5f6f05778d2ca44b577b50cd19a6646b570
$ git push -q
```

Create a submodule from the subpackage `b`. Note at this point, only `github.com/go-modules-by-example/cyclic
` can be resolved over
the internet; `github.com/go-modules-by-example/cyclic/b
` only exists locally. This will also be the case for any CI run before our
next commit is referenced by `master` on our remote. Hence we temporarily use relative `replace` directives for the
dependencies `a -> b` and `b -> a`. However, because of [#26241](https://github.com/golang/go/issues/26241), we cannot
currently use a `replace` directive to satisfy the dependency `a -> b`. So for now this next step is pushed in a "broken"
state; we don't add relative `replace` directives for either dependency. This step will be updated once
[#26241](https://github.com/golang/go/issues/26241) is fixed.

```
$ cd /home/gopher/scratchpad/cyclic/b
$ go mod init github.com/go-modules-by-example/cyclic/b
go: creating new go.mod: module github.com/go-modules-by-example/cyclic/b
```

Commit and push:

```
$ cd /home/gopher/scratchpad/cyclic
$ git add -A
$ git commit -q -am "Commit 2: create github.com/go-modules-by-example/cyclic/b"
$ git rev-parse HEAD
3e716c4a30978493e46a9d1a3a1f08f24d8f5c8b
$ git push -q
```

Until [#26241](https://github.com/golang/go/issues/26241) is merged, this is where the mutual dependency gets created:

```
$ go test -v ./...
go: finding github.com/go-modules-by-example/cyclic/b latest
go: downloading github.com/go-modules-by-example/cyclic/b v0.0.0-20190308213354-3e716c4a3097
go: extracting github.com/go-modules-by-example/cyclic/b v0.0.0-20190308213354-3e716c4a3097
?   	github.com/go-modules-by-example/cyclic/a	[no test files]
$ cd /home/gopher/scratchpad/cyclic/b
$ go test -v ./...
go: finding github.com/go-modules-by-example/cyclic/a latest
go: finding github.com/go-modules-by-example/cyclic latest
go: downloading github.com/go-modules-by-example/cyclic v0.0.0-20190308213354-3e716c4a3097
go: extracting github.com/go-modules-by-example/cyclic v0.0.0-20190308213354-3e716c4a3097
=== RUN   TestUsingA
Here is A: B
--- PASS: TestUsingA (0.00s)
PASS
ok  	github.com/go-modules-by-example/cyclic/b	(cached)
```

List the dependencies of `github.com/go-modules-by-example/cyclic
`:

```
$ cd /home/gopher/scratchpad/cyclic
$ go list -m all
github.com/go-modules-by-example/cyclic
github.com/go-modules-by-example/cyclic/b v0.0.0-20190308213354-3e716c4a3097
```

List the dependencies of `github.com/go-modules-by-example/cyclic/b
`:

```
$ cd /home/gopher/scratchpad/cyclic/b
$ go list -m all
github.com/go-modules-by-example/cyclic/b
github.com/go-modules-by-example/cyclic v0.0.0-20190308213354-3e716c4a3097
```

Commit the mutual dependency:

```
$ cd /home/gopher/scratchpad/cyclic
$ git add -A
$ git commit -q -am "Commit 3: the mutual dependency"
$ git rev-parse HEAD
01347b0ed7a65e3ad18c2965b364557c216f8ad2
$ git push -q
```

### Version details

```
go version go1.12 linux/amd64
```

<!-- END -->
