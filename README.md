# Tilt API Style Guide

## Table of Contents

* [General Perl](#general-perl) – Best practices for [Perl](https://www.perl.org), our API's main language
* [Moose](#moose) – Best practices for [Moose](https://metacpan.org/pod/Moose), our object system
* [DBIC](#dbic) – Best practices for [DBIx::Class](https://metacpan.org/pod/DBIx::Class), our ORM library
* [Tests](#tests) – Best practices for our `prove` and `TAP` based test harnesses
* [Postgres](#postgres) – Best practices for [PostgreSQL](http://www.postgresql.org) and [PL/pgSQL](http://www.postgresql.org/docs/9.4/static/plpgsql.html), our RDBMS

## General Perl

### Always use Tilt::Core

[Tilt::Core](https://github.com/Crowdtilt/crowdtilt-internal-api/blob/dev/lib/Tilt/Core.pm) gives us a robust set of baseline Perl modules, pragmas, and features, and we should always be importing this module first.

```perl
use Tilt::Core;
```

### Favor strict mode

It is better to import `Tilt::Core` with the `:strict_fp` flag instead of the default "lax" mode, and this utilizes [Function::Parameters :strict](https://metacpan.org/pod/Function::Parameters#Keyword) under the hood (enables argument checks so that an improper number of arguments will throw an error). We are still in the process of migrating our existing modules from "lax" to "strict", but we should follow this pattern for all greenfield development.

```perl
use Tilt::Core qw(:strict_fp);
```

### Favor newer Tilt::* imports

It is clearer to import the newer [Tilt::Error](https://github.com/Crowdtilt/crowdtilt-internal-api/blob/dev/lib/Tilt/Error.pm) and [Tilt::Util](https://github.com/Crowdtilt/crowdtilt-internal-api/blob/dev/lib/Tilt/Util.pm) modules over `Crowdtilt::Internal::Error` and the `Crowdtilt::Internal::Util::*` namespace.

```perl
use Tilt::Error;
use Tilt::Util qw(BankAccount Campaign User);
```

### Use structured exception handling

We should utilize `try`/`catch`/`finally` over Perl's magical `$@` exception handling. It is also best practice to catch specific, typed exception classes before the stringified "catch-all" exception.

```perl
try {
    my $result = 1/0;
} catch (CustomExceptionClass $exc) {
    print "Exception: $exc";
    die $exc;
} catch ($err) {
    print "Something else went wrong: $err";
} finally {
    do_cleanup();
}
```

### Avoid the Smartmatch operator

Smart matching `~~` is [considered deprecated](http://search.cpan.org/dist/perl-5.18.0/pod/perldelta.pod#The_smartmatch_family_of_features_are_now_experimental) and carries with it a number of issues and inconsistencies, so use the `|in|` operator instead to check whether an element is present inside of an ArrayRef.

```perl
$match = 5 |in| [ 5, 6, 7, 8 ];
```

### Avoid the Enterprise operator

Instead of using the syntactically noisy [Enterprise operator](https://metacpan.org/pod/distribution/perlsecret/lib/perlsecret.pod#Enterprise) `()x!!` to conditionally construct our HashRefs, we have the clearer `maybe` and `provided` that are in more familiar English.

```perl
my $person = {
    maybe name => $name,
    maybe age  => $age,
    provided $inches > 200, height => $inches,
    provided $pounds < 200, weight => $pounds,
};
```

### Keep it under 80 characters

Do not exceed 80 characters per line of code. There are exceptions, but generally if one cannot fit everything onto one line, it is best practice to break up the long statement into multiple lines.

```perl
return $self->contributions->search({
    status  => [qw(status_one status_two status_three)],
    user_id => $self->user_id,
})->get_column('netamount')->sum // 0;
```

### Follow snake_case convention

Make sure you are naming methods and variables using the [snake_case](https://en.wikipedia.org/wiki/Snake_case) naming convention and not `camelCase`, `TitleCase`, or some other naming convention.

```perl
fun calculate_gross_profit_margin(Int $gross_profit, Int $total_revenue) {
    return $gross_profit / $total_revenue;
}
```

### Vertically align hashes

For our Hash and HashRef declerations, we indent key/value pairs so that they are aligned on the [Fat Comma](https://metacpan.org/pod/distribution/perl/pod/perlop.pod#Comma-Operator) `=>` operator to enhance readability.

```perl
my $vars = {
    api_org             => $campaign->org,
    from_email          => _cdata_decode($from->email),
    body                => _cdata_decode($body),
    subject             => _cdata_decode($subject),
    admin_name          => _cdata_decode($admin->full_name),
    campaign_short_link => $campaign->get_short_link,
    campaign_title      => _cdata_decode($campaign->title),
    reply_email         => $admin->email,
};
```

### Prefer vertically aligned assignments

This one is not a hard requirement, but it can enhance readability for long stretches of [Assignment](https://metacpan.org/pod/distribution/perl/pod/perlop.pod#Assignment-Operators) `=` operators on variables.

```perl
my $from    = smart_rset('User')->on_crowdtilt->find($msg->{from_id});
my $to      = smart_rset('User')->on_crowdtilt->find($msg->{to_id});
my $body    = $msg->{body};
my $subject = $msg->{subject};
my $locale  = locale_from_user_country_code $to->country_code;
```

## Moose

### Use Modern MooseX::* libraries

We should always use [MooseX::Modern](https://metacpan.org/pod/MooseX::Modern) or [MooseX::Modern::Role](https://metacpan.org/pod/MooseX::Modern::Role) when defining classes and roles. These libraries import several useful attribute shortcuts that align with our best practices. They also [automatically clean the namespace](https://metacpan.org/pod/namespace::autoclean) in which the Moose class is defined, so that only instance-based (OO) methods are exported.

```perl
use MooseX::Modern;

# Attributes and methods here
```

### Always make classes immutable

We should always be making classes immutable at the very bottom of our class definitions. Even if [MOP](https://metacpan.org/pod/Class::MOP) (Meta-Object Protocol, or metaprogramming) is to be utilized, it is still preferable to explicitly mark a class as mutable, and then back to immutable once those MOP operations are complete.

```perl
# Class definition

__PACKAGE__->meta->make_immutable;

1;
```

### Never override new

We should never override `new`. Instead, we use `BUILD` or `BUILDARGS` to hook into object construction.

The `BUILDARGS` method is called *before* an object has been created, and the primary purpose of `BUILDARGS` is to allow a class to accept something other than named arguments.

The `BUILD` method is called *after* the object is constructed, but before it is returned to the caller, and provides an opportunity to check the object state as a whole. This is a good place to put logic that cannot be expressed as a type constraint on a single attribute.

[Here](https://metacpan.org/pod/distribution/Moose/lib/Moose/Cookbook/Basics/Person_BUILDARGSAndBUILD.pod#SYNOPSIS) is an excellent example in the Moose cookbook for using both methods.

### Enforce immutable state

We should enforce [read-only attributes](https://metacpan.org/pod/distribution/Moose/lib/Moose/Manual/Attributes.pod#Read-write-vs.-read-only) for immutable state. All attributes are defaulted to read-only thanks to an [attribute shortcut](https://metacpan.org/pod/MooseX::HasDefaults::RO) that we use internally, so the `ro` keyword is not explicitly needed in new attribute definitions.
    
If an attribute *must* be read-write, it is preferable to define the writer as a separate private method with the `rwp` shortcut, as narrower APIs are much easier to maintain.

```perl
# Writer automatically installed as `_set_pizza`
has pizza => (
    is  => 'rwp',
    isa => 'Pizza',
);
```

### Encourage laziness

We encourage `lazy` for calculated attributes unless those attributes are `required` or have trivial defaults. Setting `is => lazy` automatically install a builder for the attribute, and is the recommended approach over an anonymous default sub.

```perl
has size => (
    is      => 'lazy',
    isa     => 'Str',
    builder => method {
        return (qw/small medium large/)[ int(rand 3) ];
    },
);
```

## DBIC

## Tests

### Unit test with t::lib::Unit

For adding additional unit test coverage to existing code, or for new greenfield deevelopment, make sure to use `t::lib::Unit` to take advantage of the most streamlined imports and mocking facilities, and minimize boilerplate in test code.

```perl
use t::lib::Unit;
```

### Use MO and MM for mocking

The `MO` function is a handy shortcut to quickly creating [Test::MockObject](https://metacpan.org/pod/Test::MockObject) objects, and `MM` is a clean, scoped wrapper around [Sub::Override](https://metacpan.org/pod/Sub::Override), both of which can save a lot of time mocking common objects and interfaces.

```perl
# Mock Object
my $mock_user = MO(
    country_code      => 'US',
    email             => 'test@email.com',
    guid              => 'USR12345',
    registration_date => fun { DateTime->now->iso8601 },
);

# Mock Module
my $mock_request_id = MM('Crowdtilt::Internal::API',
    get_request_id => 'fake_request_id',
);
```

## Postgres
