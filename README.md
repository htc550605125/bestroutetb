# Best Route Table [![NPM version](https://badge.fury.io/js/bestroutetb.svg)](http://badge.fury.io/js/bestroutetb) [![Dependency](https://david-dm.org/ashi009/bestroutetb.svg)](https://david-dm.org/ashi009/bestroutetb)

Inspired by [chnroutes][chnroutes].

This project aimed to generate the smallest route table,
while preserves the minimalist requirements that IPs of
specified countries or subnets will be routed to a
specified gateway (default or VPN).

Generally speaking, the generated route table is at least
70% smaller than chnroutes's.


## Objective

I started this project as a result of the huge route table
generated by *chnroutes* didn't fit into my router.

The route table took almost 4 minutes to load up, and it cannot be
put into OpenVPN's configuration file, due to my service
provider pushed `ping-reset 60` to the client, reseted
OpenVPN before route table being loaded up.

Therefore I decided to minimize the route table.


## How efficient it is?

For an example, a route table that route all IPs in China to
default gateway, and US, GB, Japan, Hong kong administered
IPs to VPN gateway, only need 1546 routing directives,
while *chnroutes* needs 4953 routing directives (based on 11/21/2014 data).

Which is almost **70% smaller**. And if route US address to VPN only,
the route table has only **105 directives**, which is about only
2% of original size.

On Linux system, which usese [TRASH][trash] structure to store
route table, a route lookup operation expected to access
memory O(loglog _n_) times. Using *bestroutetb* instead of *chnroutes*,
will reduce route table look up at least 0.01 times expectedly.
However, this solution does reduce the route table size for 70% by
assuming TRASH structure is implemented with a small overhead
hash implementation.  It is therefore that a perfect approach for those
routers with low free memory.


## How it works

Unlike *chnroutes* which will generate a route table that
route all subnets of China to ISP gateway, while other route to VPN gateway.
This project divides IPs in three groups. First group is guaranteed
to be routed to default gateway, Second group is guaranteed to be
routed to VPN gateway. And the last group will be dynamically assigned
to one of the gateways, in a manner that will generate
the smallest route table.

To achieve this goal, this project using dynamic programming
algorithm to find the most optimized route table.

We can prove that, the generated route table is the smallest
one based on the given restrictions.

[For further detail][blog].


## How to use

### Install

This project requires [node.js][nodejs] to run.

If you are using OS X, install node.js with homebrew:

    $ brew install nodejs

From NPM for use as a command line app:

    $ npm install -g bestroutetb

From NPM for programmatic use:

    $ npm install bestroutetb

From Git:

    $ git clone https://github.com/ashi009/bestroutetb.git
    $ cd bestroutetb
    $ npm link .

### Usage

    $ bestroutetb [options]

### Options

#### Route

    --route.net=<spec>
    --route.vpn=<spec>

Subnets that should be routed to ISP or VPN gateway.

**spec** should be a list of two-letter country initials (eg. US), subnet (eg.
`8.0.0.0/8`), and host (eg. `123.123.123.123`), concatenated with comma (`,`).

_NOTE:_ You could also use multiple `--route.*` arguments to construct the list.

#### Output Profile

    -p <profile>, --profile=<profile>

Built-in profiles are `custom`, `iproute`, `json` and `openvpn`.

#### Output

    -o <path>, --output=<path>

Output file path.

_NOTE:_ Some profiles may generate multiple files. In which case you need to
specify path to a directory (eg. `-o output/`), and you may also specify the
prefix and the desired extension for the output file (eg. `-o output/ip-.sh`).

    -f, --force

Force to overwrite existing files.

#### Report

    -r <path>, --report=<path>

Generate a report in CSV format and save to given path. Here's an example:

Country | net | vpn
--- | ---:| ---:
AD | 0 | 33792
AE | 512 | 3738752
AF | 125952 | 3072

#### Output format

_NOTE:_ These formats applies when `--profile=custom`, or in case you want
to override default settings in profile.
All strings will be outputted without adding new line (`\n`).
Thus, it would be favorable for you to add them in the string.  For zsh, bash
and some other shells, you could use `$'line\n'` to include a escaped character.

    --header=<string>
    --footer=<string>

Header and Footer of the output file.

    --rule-format=<string>

String used to format a rule.

You may use `%prefix`, `%mask`, `%length` and `%gateway` or `%gw` in the string.

- `%prefix` is the prefix of the subnet (eg. `14.0.0.0`).
- `%mask` is the mask of the subnet (eg. `255.0.0.0`).
- `%length` is the length of the mask (eg. `8`).
- `%gateway` is the routing destination of the subnet (eg. `net` and `vpn`).
- `%gw` is the customized destination (eg. `pppoe` and `tun0`), which is set with
  `--gateway.net` and `--gateway.vpn`.

<!-- -->

    --gateway.net=<string>
    --gateway.vpn=<string>

Define substitutes for `%gw` in rule format.

    --[no-]default-gateway

Whether to output directive for default route (`0.0.0.0/0`), which would be
outputted by default.

    --[no-]group-gateway

Whether to group rules by gateway.

    --group-header=<string>
    --group-footer=<string>

Header and Footer of each group.

You may include `%name` in the string.

- `%name` is the customized group name (eg. `wan` and `vpn`), which is set with
  `--group-name.net` and `--group-name.vpn`.

<!-- -->

    --group-name.net=<string>
    --group-name.vpn=<string>

Define substitutes for `%name` in group header and group footer.

#### Update

    --[no-]update

Force update delegation data, or force to use stale data.

#### Config

    -c <path>, --config=<path>

Configuration file path.

#### Logging

    -v, --verbose

Verbose output level. Use `-vvvv` for debug.

    -s, --silent

Silent mode, which will suppress any output.

#### Help

    -h, --help

Show help.

#### Version

    -V, --version

Show version number.

### Examples

    $ bestroutetb --route.vpn=us -p json -o routes.json

[chnroutes]: https://github.com/fivesheep/chnroutes
[wiki]: https://github.com/ashi009/bestroutetb/wiki/%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E
[trash]: http://www.nada.kth.se/~snilsson/publications/TRASH/trash.pdf
[blog]: http://ashi009.tumblr.com/post/36581070478/vpn
[nodejs]: http://nodejs.org
[wget]: http://www.gnu.org/software/wget/
