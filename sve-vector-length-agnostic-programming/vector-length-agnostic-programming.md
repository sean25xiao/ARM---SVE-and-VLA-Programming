# Vector Length Agnostic Programming

> The goal of SVE is to allow the same program binary to be run on any implementation of the architecture, which might implement different vector lengths

> The SVE features requires a new programming style, called Vector Length Agnostic \(VLA\).

For example, we want to do vector-vector-add between two vectors b\[ \] and c\[ \], and then store the result in a\[ \]. There are three versions of these programs:

1. Traditional method
2. Programs on ARM Neon
3. Programs on SVE



