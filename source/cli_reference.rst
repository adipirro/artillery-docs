CLI Reference
*************

The ``artillery`` command has several sub-commands:
::

  Usage: artillery [options] [command]


  Commands:

    run [options] <script>  Run a test script. Example: `artillery run benchmark.json`
    quick [options] <url>   Run a quick test without writing a test script
    report <file>           Create a report from a JSON file created by "artillery run"
    convert <file>          Convert JSON to YAML and vice versa
    dino [options]          Show dinosaur of the day

  Options:

    -h, --help     output usage information
    -V, --version  output the version number


Some commands take further options:

``run``
#######

Options for `run`:
::

  Usage: run [options] <script>

  -h, --help                output usage information
  -t, --target <url>        Set target URL
  -p, --payload <path>      Set payload file (CSV)
  -o, --output <path>       Set file to write stats to (will output to stdout by default)
  -k, --insecure            This option explicitly allows Artillery to perform "insecure" TLS connections, for testing with self-signed certificates
  -e, --environment <name>  Specify the environment to be used
  -q, --quiet               Do not print anything to stdout


``quick``
#########

Use this command to run a quick test without needing to write a scenario. Example:

``artillery quick -d 60 -r 10 -k https://myapp.dev``

Options for `quick`:
::
  
  Usage: quick [options] <url>

  -h, --help                   output usage information
  -r, --rate <number>          New arrivals per second
  -c, --count <number>         Fixed number of arrivals
  -d, --duration <seconds>     Duration of the arrival phase
  -n, --num <number>           Number of requests each new arrival will send
  -p, --payload <string>       Set POST payload
  -t, --content-type <string>  Set content-type (defaults to application/json)
  -o, --output <path>          Set file to write stats to (will output to stdout by default)
  -k, --insecure               This  option explicitly allows Artillery to perform "insecure" TLS connections, for testing withself-signed certificates
  -q, --quiet                  Do not print anything to stdout
