[![PyPI](https://img.shields.io/pypi/v/json-repair)](https://pypi.org/project/json-repair/)
![Python version](https://img.shields.io/badge/python-3.8+-important)
[![PyPI downloads](https://img.shields.io/pypi/dm/json-repair)](https://pypi.org/project/json-repair/)
[![Github Sponsors](https://img.shields.io/github/sponsors/mangiucugna)](https://github.com/sponsors/mangiucugna)

This simple package can be used to fix an invalid json string. To know all cases in which this package will work, check out the unit test.

Inspired by https://github.com/josdejong/jsonrepair

---
# How to cite
If you are using this library in your academic work (as I know many folks are) please find the BibTex here:

    @software{Baccianella_JSON_Repair_-_2024,
        author = {Baccianella, Stefano},
        month = aug,
        title = {{JSON Repair - A python module to repair invalid JSON, commonly used to parse the output of LLMs}},
        url = {https://github.com/mangiucugna/json_repair},
        version = {0.28.3},
        year = {2024}
    }

Thank you for citing my work and please send me a link to the paper if you can!

---
# Offer me a beer
If you find this library useful, you can help me by donating toward my monthly beer budget here: https://github.com/sponsors/mangiucugna

---

# Demo
If you are unsure if this library will fix your specific problem, or simply want your json validated online, you can visit the demo site on GitHub pages: https://mangiucugna.github.io/json_repair/

---

# Motivation
Some LLMs are a bit iffy when it comes to returning well formed JSON data, sometimes they skip a parentheses and sometimes they add some words in it, because that's what an LLM does.
Luckily, the mistakes LLMs make are simple enough to be fixed without destroying the content.

I searched for a lightweight python package that was able to reliably fix this problem but couldn't find any.

*So I wrote one*

# How to use
    from json_repair import repair_json

    good_json_string = repair_json(bad_json_string)
    # If the string was super broken this will return an empty string

You can use this library to completely replace `json.loads()`:

    import json_repair

    decoded_object = json_repair.loads(json_string)

or just

    import json_repair

    decoded_object = json_repair.repair_json(json_string, return_objects=True)

### Avoid this antipattern
Some users of this library adopt the following pattern:

    obj = {}
    try:
        obj = json.loads(string)
    except json.JSONDecodeError as e:
        obj = json_repair.loads(string)
        ...

This is wasteful because `json_repair` will already verify for you if the JSON is valid, if you still want to do that then add `skip_json_loads=True` to the call as explained the section below.

### Read json from a file or file descriptor

JSON repair provides also a drop-in replacement for `json.load()`:

    import json_repair

    try:
        file_descriptor = open(fname, 'rb')
    except OSError:
        ...

    with file_descriptor:
        decoded_object = json_repair.load(file_descriptor)

and another method to read from a file:

    import json_repair

    try:
        decoded_object = json_repair.from_file(json_file)
    except OSError:
        ...
    except IOError:
        ...

Keep in mind that the library will not catch any IO-related exception and those will need to be managed by you

### Performance considerations
If you find this library too slow because is using `json.loads()` you can skip that by passing `skip_json_loads=True` to `repair_json`. Like:

    from json_repair import repair_json

    good_json_string = repair_json(bad_json_string, skip_json_loads=True)

I made a choice of not using any fast json library to avoid having any external dependency, so that anybody can use it regardless of their stack.

Some rules of thumb to use:
- Setting `return_objects=True` will always be faster because the parser returns an object already and it doesn't have serialize that object to JSON
- `skip_json_loads` is faster only if you 100% know that the string is not a valid JSON
- If you are having issues with escaping pass the string as **raw** string like: `r"string with escaping\""`

## Adding to requirements
**Please pin this library only on the major version!**

We use TDD and strict semantic versioning, there will be frequent updates and no breaking changes in minor and patch versions.
To ensure that you only pin the major version of this library in your `requirements.txt`, specify the package name followed by the major version and a wildcard for minor and patch versions. For example:

    json_repair==0.*

In this example, any version that starts with `0.` will be acceptable, allowing for updates on minor and patch versions.

# How it works
This module will parse the JSON file following the BNF definition:

    <json> ::= <primitive> | <container>

    <primitive> ::= <number> | <string> | <boolean>
    ; Where:
    ; <number> is a valid real number expressed in one of a number of given formats
    ; <string> is a string of valid characters enclosed in quotes
    ; <boolean> is one of the literal strings 'true', 'false', or 'null' (unquoted)

    <container> ::= <object> | <array>
    <array> ::= '[' [ <json> *(', ' <json>) ] ']' ; A sequence of JSON values separated by commas
    <object> ::= '{' [ <member> *(', ' <member>) ] '}' ; A sequence of 'members'
    <member> ::= <string> ': ' <json> ; A pair consisting of a name, and a JSON value

If something is wrong (a missing parantheses or quotes for example) it will use a few simple heuristics to fix the JSON string:
- Add the missing parentheses if the parser believes that the array or object should be closed
- Quote strings or add missing single quotes
- Adjust whitespaces and remove line breaks

I am sure some corner cases will be missing, if you have examples please open an issue or even better push a PR

# How to develop
Just create a virtual environment with `requirements.txt`, the setup uses [pre-commit](https://pre-commit.com/) to make sure all tests are run.

Make sure that the Github Actions running after pushing a new commit don't fail as well.

# How to release
You will need owner access to this repository
- Edit `pyproject.toml` and update the version number appropriately using `semver` notation
- **Commit and push all changes to the repository before continuing or the next steps will fail**
- Run `python -m build`
- Create a new release in Github, making sure to tag all the issues solved and contributors. Create the new tag, same as the one in the build configuration
- Once the release is created, a new Github Actions workflow will start to publish on Pypi, make sure it didn't fail
---
# Repair JSON in other programming languages
- Typescript: https://github.com/josdejong/jsonrepair
- Go: https://github.com/RealAlexandreAI/json-repair
- Ruby: https://github.com/sashazykov/json-repair-rb
---
## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=mangiucugna/json_repair&type=Date)](https://star-history.com/#mangiucugna/json_repair&Date)
