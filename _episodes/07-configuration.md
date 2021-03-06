---
title: "Nextflow configuration"
teaching: 10
exercises: 10
questions:
- "How do you configure Nextflow?"
- "How do you write a Nextflow configuration file?"
objectives:
- "Understand who Nextflow is configured."
- "Create a Nextflow configuration file."
keypoints:
- "Nextflow configuration can be managed using a nextflow.config."
- "Nextflow configuration are simple text files ontaining a set of properties defined using the syntax"
---


## Nextflow configuration

A key Nextflow feature is the ability to decouple the workflow implementation by the configuration setting required by the underlying execution platform.

This enable portable deployment without the need to modify the application code.

## Configuration file

When a pipeline script is launched Nextflow looks for a file named `nextflow.config` in the current directory and in the script base directory 
(if it is not the same as the current directory). Finally it checks for the file `$HOME/.nextflow/config`.

When more than one on the above files exist they are merged, so that the settings in the first override the same ones that may appear in the second one, 
and so on.

The default config file search mechanism can be extended proving an extra configuration file by using the command line option `-c <config file>`.

## Config syntax

A Nextflow configuration file is a simple text file containing a set of properties defined using the syntax:

`name = value`

Please note, string values need to be wrapped in quotation characters while numbers and boolean values (true, false) do not. 
Also note that values are typed, meaning for example that, `1` is different from `'1'`, 
since the first is interpreted as the number one, while the latter is interpreted as a string value.

### Config variables

Configuration properties can be used as variables in the configuration file itself, by using the usual `$propertyName` or `${expression}` syntax.

~~~
propertyOne = 'world'
anotherProp = "Hello $propertyOne"
customPath = "$PATH:/my/app/folder"
~~~
{: .source}

In the configuration file it’s possible to access any variable defined in the host environment such as `$PATH`, `$HOME`, `$PWD`, etc.


### Config comments

Configuration files use the same conventions for comments used in the Nextflow script:

~~~
// comment a single config file

/*
   a comment spanning
   multiple lines
 */
~~~

### Config scopes

Configuration settings can be organized in different scopes by dot prefixing the property names with a scope identifier or grouping the properties 
in the same scope using the curly brackets notation `{}`. This is shown in the following example:

~~~
alpha.x  = 1
alpha.y  = 'string value..'

beta {
    p = 2
    q = 'another string ..'
}
~~~
{: .source}
### Config params

The scope `param`s allows the definition of workflow parameters that overrides the values defined in the main workflow script.

This is useful to consolidate one or more execution parameters in a separate file.

~~~
// config file
params.foo = 'Bonjour'
params.bar = 'le monde!'

// workflow script
params.foo = 'Hello'
params.bar = 'world!'

// print the both params
println "$params.foo $params.bar"
Exercise
Save the first snippet as nextflow.config and the second one as params.nf. Then run:

nextflow run params.nf
Execute is again specifying the foo parameter on the command line:

nextflow run params.nf --foo Hola
Compare the result of the two executions.
~~~~
{: .source}

### Config env

The `env` scope allows the definition one or more variable that will be exported in the environment where the workflow tasks will be executed.

~~~
env.ALPHA = 'some value'
env.BETA = "$HOME/some/path"
~~~
{: .source}

Exercise
Save the above snippet a file named my-env.config. The save the snippet below in a file named foo.nf:
~~~
process foo {
  echo true
  '''
  env | egrep 'ALPHA|BETA'
  '''
}
~~~
{: .source}

Finally executed the following command:

~~~
nextflow run foo.nf -c my-env.config
~~~~
{: .source}

### Config process

The `process` directives allow the specification of specific settings for the task execution such as 
`cpus`, `memory`, `container` and other resources in the pipeline script.

This is useful specially when prototyping a small workflow script.

However it’s always a good practice to decouple the workflow execution logic from the process configuration settings, 
i.e. it’s strongly suggested to define the process settings in the workflow configuration file instead of the workflow script.
The process configuration scope allows the setting of any process directives in the Nextflow configuration file. For example:

~~~
process {
    cpus = 10
    memory = 8.GB
    container = 'biocontainers/bamtools:v2.4.0_cv3'
}
~~~
{: .source}

The above config snippet defines the `cpus`, `memory` and `container` directives for **all** processes in your workflow script.

The `process` selector can be used to apply the configuration to a specific process or group of processes (discussed later).

Memory and time duration unit can be specified either using a string based notation in which the digit(s) and the unit can be separated by a blank or 
by using the numeric notation in which the digit(s) and the unit are separated by a dot character and it’s not enclosed by quote characters.
String syntax	Numeric syntax	Value

~~~
'10 KB'

10.KB

10240 bytes

'500 MB'

500.MB

524288000 bytes

'1 min'

1.min

60 seconds

'1 hour 25 sec'

-

1 hour and 25 seconds
~~~

The syntax for setting process directives in the configuration file requires = ie. assignment operator, instead it should not be used when setting process directives in the workflow script.
This important especially when you want to define a config setting using a dynamic expression using a closure. For example:

~~~
process {
    memory = { 4.GB * task.cpus }
}
~~~

Directives that requires more than one value, e.g. pod, in the configuration file need to be expressed as a map object.

~~~
process {
    pod = [env: 'FOO', value: '123']
}
~~~


Finally directives that allows to be repeated in the process definition, in the configuration files need to be defined as a list object. For example:

~~~
process {
    pod = [ [env: 'FOO', value: '123'],
            [env: 'BAR', value: '456'] ]
}
~~~
{: .source}

### Config Docker execution

The container image to be used for the process execution can be specified in the `nextflow.config` file:

~~~
process.container = 'nextflow/rnaseq-nf'
docker.enabled = true
~~~
{: .source}

The use of the unique SHA256 image ID guarantees that the image content do not change over time

~~~
process.container = 'nextflow/rnaseq-nf@sha256:aeacbd7ea1154f263cda972a96920fb228b2033544c2641476350b9317dab266'
docker.enabled = true
~~~

{: .source}

### Config Singularity execution

The run the workflow execution with a Singularity container provide the container image file path in the Nextflow config file using the container directive:

~~~
process.container = '/some/singularity/image.sif'
singularity.enabled = true
~~~
{: .source}

The container image file must be an absolute path i.e. it must start with a /.

The following protocols are supported:

library:// download the container image from the Singularity Library service.

shub:// download the container image from the Singularity Hub.

docker:// download the container image from the Docker Hub and convert it to the Singularity format.

docker-daemon:// pull the container image from a local Docker installation and convert it to a Singularity image file.

Specifying a plain Docker container image name, Nextflow implicitly download and converts it to a Singularity image when the Singularity execution is enabled. 
For example:

~~~
process.container = 'nextflow/rnaseq-nf'
singularity.enabled = true
~~~
{: .source}

The above configuration instructs Nextflow to use Singularity engine to run your script processes. 
The container is pulled from the Docker registry and cached in the current directory to be used for further runs.

Alternatively if you have a Singularity image file, its location absolute path can be specified as the container name either using the 
`-with-singularity` option or the process.container setting in the config file.

Try to run the script as shown below:

~~~
nextflow run script7.nf
~~~

Note: Nextflow will pull the container image automatically, it will require a few seconds depending the network connection speed.

### Config Conda execution

The use of a Conda environment can also be provided in the configuration file adding the following setting in the `nextflow.config` file:

~~~
process.conda = "/home/ubuntu/miniconda2/envs/nf-tutorial"
~~~
{: .source}

You can either specify the path of an existing Conda environment directory or the path of Conda environment `YAML` file.













{% include links.md %}
