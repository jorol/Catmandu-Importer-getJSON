# NAME

Catmandu::Importer::getJSON - load JSON-encoded data from a server using a GET HTTP request

# STATUS

[![Build Status](https://travis-ci.org/nichtich/Catmandu-Importer-getJSON.png)](https://travis-ci.org/nichtich/Catmandu-Importer-getJSON)
[![Coverage Status](https://coveralls.io/repos/nichtich/Catmandu-Importer-getJSON/badge.png)](https://coveralls.io/r/nichtich/Catmandu-Importer-getJSON)
[![Kwalitee Score](http://cpants.cpanauthors.org/dist/Catmandu-Importer-getJSON.png)](http://cpants.cpanauthors.org/dist/Catmandu-Importer-getJSON)

# SYNOPSIS

The following three examples are equivalent:

    Catmandu::Importer::getJSON->new(
        file => \"http://example.org/alice.json\nhttp://example.org/bob.json"
    )->each(sub { my ($record) = @_; ... );

    Catmandu::Importer::getJSON->new(
        url  => "http://example.org",
        file => \"/alice.json\n/bob.json"
    )->each(sub { my ($record) = @_; ... );

    Catmandu::Importer::getJSON->new(
        url  => "http://example.org/{name}.json",
        file => \"{\"name\":\"alice\"}\n{\"name\":\"bob\"}"
    )->each(sub { my ($record) = @_; ... );

For more convenience the [catmandu](https://metacpan.org/pod/catmandu) command line client can be used:

    echo http://example.org/alice.json | catmandu convert getJSON to YAML
    catmandu convert getJSON --from http://example.org/alice.json to YAML
    catmandu convert getJSON --dry 1 --url http://{domain}/robots.txt < domains

# DESCRIPTION

This [Catmandu::Importer](https://metacpan.org/pod/Catmandu::Importer) performs a HTTP GET request to load JSON-encoded
data from a server. The importer expects a line-separated input. Each line
corresponds to a HTTP request that is mapped to a JSON-record on success. The
following input formats are accepted:

- plain URL

    A line that starts with "`http://`" or "`https://`" is used as plain URL.

- URL path

    A line that starts with "`/`" is appended to the configured **url** parameter.

- variables

    A JSON object with variables to be used with an URL template or as HTTP query
    parameters. For instance the input line `{"name":"Karl Marx"}` with URL
    `http://api.lobid.org/person` or the input line
    `{"entity":"person","name":"Karl Marx"}` with URL template
    `http://api.lobid.org/{entity}{?id}{?name}{?q}` are both expanded to
    [http://api.lobid.org/person?name=Karl+Marx](http://api.lobid.org/person?name=Karl+Marx).

If the JSON data returned in a HTTP response is a JSON array, its elements are
imported as multiple items. If a JSON object is returned, it is imported as one
item.

# CONFIGURATION

- url

    An [URI](https://metacpan.org/pod/URI) or an URI templates ([URI::Template](https://metacpan.org/pod/URI::Template)) as defined by
    [RFC 6570](http://tools.ietf.org/html/rfc6570) to load JSON from. If no **url**
    is configured, plain URLs must be provided as input or option `from` must be
    used instead.

- from

    A plain URL to load JSON without reading any input lines.

- timeout / agent / proxy / headers

    Optional HTTP client settings.

- client

    Instance of a [Furl](https://metacpan.org/pod/Furl) HTTP client to perform requests with.

- dry

    Don't do any HTTP requests but return URLs that data would be queried from.

- file / fh

    Input to read lines from (see [Catmandu::Importer](https://metacpan.org/pod/Catmandu::Importer)). Defaults to STDIN.

- fix

    An optional fix to be applied on every item (see [Catmandu::Fix](https://metacpan.org/pod/Catmandu::Fix)).

- wait

    Number of seconds to wait between requests.

- cache

    Cache JSON response of URLs to not request the same URL twice. HTTP error
    codes in the 4xx range (e.g. 404) are also cached but 5xx errors are not.

    The value of this option can be any objects that implements method `get` and
    `set` (e.g. `CHI`), an existing directory for file caching, a true value to
    enable global in-memory-caching, or a false value to disable caching (default).

    File caching uses file names based on MD5 of an URL so for instance
    `http://example.org/` is cached as `4389382917e51695b759543fdfd5f690.json`.

- warn

    Show error messages on the standard error.

# METHODS

## time

Returns the UNIX timestamp right before the last request. This can be used for
instance to add timestamps or the measure how fast requests were responded.

## construct\_url( \[ $base\_url, \] $vars\_url\_or\_path )

Returns an URL given a hash reference with variables, a plain URL or an URL
path. The optional first argument can be used to override option `url`.

    $importer->construct_url( %query_vars )
    $importer->construct_url( $importer->url, %query_vars ) # equivalent

## request($url)

Perform a HTTP GET request of a given URL including logging, caching, request
hook etc. Returns a hash/array reference or `undef`.

# EXTENDING

This importer provides two methods to filter requests and responses,
respectively. See [Catmandu::Importer::Wikidata](https://metacpan.org/pod/Catmandu::Importer::Wikidata) for an example.

## request\_hook

Gets a whitespace-trimmed input line and is expected to return an unblessed
hash reference, an URL, or undef. Errors are catched and treated equal to
undef.

## response\_hook

Gets the queried response object and is expected to return an object.

# LOGGING

URLs are emitted before each request on DEBUG log level.

# LIMITATIONS

Future versions of this module may also support asynchronous HTTP fetching
modules such as [HTTP::Async](https://metacpan.org/pod/HTTP::Async), for retrieving multiple URLs at the same time.

# SEE ALSO

[Catmandu::Fix::get\_json](https://metacpan.org/pod/Catmandu::Fix::get_json) provides this importer as fix function.

# COPYRIGHT AND LICENSE

Copyright Jakob Voß, 2014-

This library is free software; you can redistribute it and/or modify it under
the same terms as Perl itself.
