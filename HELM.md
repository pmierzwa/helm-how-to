# Helm

## [Getting started](https://helm.sh/docs/chart_template_guide/getting_started/)

Release using Helm:

``` bash
helm install <release-name> <path-to-chart>
```

Unrelease

``` bash
helm uninstall <release-name>
```

Validate
``` bash
helm install --debug --dry-run <release-name> <path-to-chart>
```

## [Built-in Objects](https://helm.sh/docs/chart_template_guide/builtin_objects/)

The following top-level objects exists:

* **Release** - This object describes the release itself

* **Values** - Values passed into the template from the `values.yaml` file and from user-supplied files. By default, `Values` is empty.

* **Chart** - The contents of the `Chart.yaml` file. Any data in `Chart.yaml` will be accessible here. For example `{{ .Chart.Name }}-{{ .Chart.Version }}` will print out the `mychart-0.1.0`.

* **Files** - This provides access to all non-special files in a chart. While you cannot use it to access templates, you can use it to access other files in the chart.

* **Capabilities** - This provides information about what capabilities the Kubernetes cluster supports.

* **Template** - Contains information about the current template that is being executed

## [Values Files](https://helm.sh/docs/chart_template_guide/values_files/)

You can pass parameters like so:

``` bash
helm install --set foo=bar <path-to-chart>
```

## [Template Functions and Pipelines](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)


You can call functions in the template directly. Such as:

``` bash
drink: {{ quote .Values.favorite.drink }}
```

which will get the value from favorite.drink, and pass it to the quote function. This could also be written as:

``` bash
drink: {{ .Values.favorite.drink | quote }}
```

Another function is the default, which can be used as such:

``` bash
drink: {{ .Values.favorite.drink | default "tea" | quote }}
```

furthermore operators (`eq`, `ne`, `lt`, `gt`, `and`, `or`) are also implemented.

## [Template Function List](https://helm.sh/docs/chart_template_guide/function_list/)


This page shows different function. Some mentioned below. The list of categories are shown below:

* Cryptographic and Security
* Date
* Dictionaries
* Encoding
* File Path
* Kubernetes and Chart
* Logic and Flow Control
* Lists
* Math
* Float Math
* Network
* Reflection
* Regular Expressions
* Semantic Versions
* String
* Type Conversion
* URL
* UUID

#### lower

``` bash
lower "HELLO"
```

The above proiduce the following:

``` bash
"hello"
```

#### contains

``` bash
contains "cat" "catch"
```

The above returns `true` because `catch` contains `cat`.

#### sha256sum

The `sha256sum` function receives a string, and computes it's SHA256 digest.

``` bash
sha256sum "Hello world!"
```

The above will compute the SHA 256 sum in an "ASCII armored" format that is safe to print.


#### date

* now
* ago
etc.

## [Flow control](https://helm.sh/docs/chart_template_guide/control_structures/)

You can use `if / else`, `with` and `range` control structures in your ConfigMap.

You can use the follwing `with` statement, to simplify your configMap:

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
```

Notice you can use `.drink` directly, instead of using `.Values.favorite.drink`.

## [Variables](https://helm.sh/docs/chart_template_guide/variables/)

You can assign variables inside configMap.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- $relname := .Release.Name -}}
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ $relname }}
  {{- end }}
```

in the code above, the `$relname` is a variable

## [Named Templates](https://helm.sh/docs/chart_template_guide/named_templates/)

You can create templates in, and re-use them in the file:

``` yaml
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```

in the above example, we see the `mychart.labels` is defined before it is used.
Instead of doing this in the same file, it is usually put it its own file: templates/_helpers.tpl. Even if the define is moved to another file, it can still be accessed in the same way, as template names are global.
Notice that in the above example, no scope is passed, so no functions can be called inside the template. To pass a scope, use `{{- template "mychart.labels" . }}`.

Note 2:
``` text
It is considered preferable to use include over template in Helm templates simply so that the output formatting can be handled better for YAML documents.
```

So including like this is better: `{{ include "mychart.app" . | indent 2 }}`, because you can indent etc.

## [Accessing Files Inside Templates](https://helm.sh/docs/chart_template_guide/accessing_files/)

It is possible to import a file that is not a template and inject its contents without sending the contents through the template renderer.

Gicven the following folder structure:
``` bash
foo/:
  foo.txt
  foo.yaml

bar/:
  bar.go
  bar.conf
  baz.yaml
```

You can include the files as such:

``` yaml
{{ range $path, $_ :=  .Files.Glob  "**.yaml" }}
      {{ $.Files.Get $path }}
{{ end }}
```

## [Creating a NOTES.txt File](https://helm.sh/docs/chart_template_guide/notes_files/)

Helms tool to provide instructions to its chart users. It will provide a print at the end of installing the chart. It is strongly recommended doing so, but not required.

## [Subcharts and Global Values](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/)

1. A subchart is considered "stand-alone", which means a subchart can never explicitly depend on its parent chart.
1. For that reason, a subchart cannot access the values of its parent.
1. A parent chart can override values for subcharts.
1. Helm has a concept of global values that can be accessed by all charts.


Subcharts lives inside the charts folder, in a subfolder called charts. As such:

``` bash
mychart
    templates
    Chart.yaml
    values.yaml
    charts
        mysubchart
            templates
            Chart.yaml
            values.yaml
```

## [The .helmignore file](https://helm.sh/docs/chart_template_guide/helm_ignore_file/)

The .helmignore file is used to specify files you don't want to include in your helm chart.

If this file exists, the helm package command will ignore all the files that match the pattern specified in the .helmignore file while packaging your application.

works as a gitignore file, but for helm.

## [Debugging Templates](https://helm.sh/docs/chart_template_guide/debugging/)

There are a few commands that can help you debug.

* `helm lint` is your go-to tool for verifying that your chart follows best practices
* `helm template --debug` will test rendering chart templates locally.
* `helm install --dry-run --debug`: We've seen this trick already. It's a great way to have the server render your templates, then return the resulting manifest file.
* `helm get manifest`: This is a good way to see what templates are installed on the server.

Notice the following:

``` yaml
apiVersion: v2
# some: problem section
# {{ .Values.foo | quote }}
```

Still works as comments, but will be rendered as: 

``` yaml
apiVersion: v2
# some: problem section
#  "bar"
```

## [Next Steps](https://helm.sh/docs/chart_template_guide/wrapping_up/)

basically just a list of resources,