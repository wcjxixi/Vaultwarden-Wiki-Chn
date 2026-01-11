# 3.Git hooks

{% hint style="success" %}
对应的[官方页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Git-hooks)
{% endhint %}

以下提交 hooks 可能对您有所帮助，请自行斟酌使用。

## `pre-commit`

```bash
#!/bin/bash

HAS_ISSUES=0
FIRST_FILE=1

colerr="$(tput setaf 9)"
colok="$(tput setaf 10)"
colreset="$(tput sgr0)"

for file in $(git diff --name-only --staged); do
    FMT_RESULT="$(rustfmt --edition 2021 --check --quiet $file 2>/dev/null || true)"
    if [ "$FMT_RESULT" != "" ]; then
        if [ $FIRST_FILE -eq 0 ]; then
            echo -n ", "
        fi
        echo -n "${colerr}${file}${colreset}"
        HAS_ISSUES=1
        FIRST_FILE=0
    fi
done

if [ $HAS_ISSUES -eq 0 ]; then
    exit 0
fi

echo "."
echo "Your code has formatting issues in the files listed above."
echo "Format your code with: ${colok}cargo fmt --all --${colreset}"
exit 1
```

## `commit-msg`

```bash
#!/bin/bash

HAS_ISSUES=0
MSG_FILE="$1"

colerr="$(tput setaf 9)"
colok="$(tput setaf 10)"
colreset="$(tput sgr0)"

MSG=`cat "$MSG_FILE" |grep -v -E "^$" |grep  -v -E "^#" |head -1`

if [ "`cat "$MSG_FILE" |grep -v -E "^$" |grep  -v -E "^#" |wc -l`" -gt 1 ]; then
	LINE2=`cat "$MSG_FILE" |grep  -v -E "^#" |head -2 |tail -1`
	if [ "$LINE2" != "" ]; then
		HAS_ISSUES=1
	fi
fi

echo $MSG |grep -qE '^(feat|fix|docs|style|refactor|perf|test|build|chore|revert|ci)(\(.+\))?!?: .*[^ ]$'

if [ "$?" != "0" ]; then
	HAS_ISSUES=1
fi

if [ $HAS_ISSUES -eq 0 ]; then
    exit 0
fi

echo "The commit message must follow the Conventional Commits specification:"
echo ""
echo "----------------------------------------------------------------------"
echo "${colok}type${colreset}[(optional scope)]: description"
echo ""
echo "[optional body]"
echo ""
echo "[optional footer(s)]"
echo "----------------------------------------------------------------------"
echo ""
echo "Where ${colok}type${colreset} must be one of:"
echo ""
echo "${colok}feat     ${colreset}| Features                 | A new feature"
echo "${colok}fix      ${colreset}| Bug Fixes                | A bug fix"
echo "${colok}docs     ${colreset}| Documentation            | Documentation only changes"
echo "${colok}style    ${colreset}| Styles                   | Changes that do not affect the meaning of the code (white-space, formatting)"
echo "${colok}refactor ${colreset}| Code Refactoring         | A code change that neither fixes a bug nor adds a feature"
echo "${colok}perf     ${colreset}| Performance Improvements | A code change that improves performance"
echo "${colok}test     ${colreset}| Tests                    | Adding missing tests or correcting existing tests"
echo "${colok}build    ${colreset}| Builds                   | Changes that affect the build system or external dependencies"
echo "${colok}ci       ${colreset}| Continuous Integrations  | Changes to our CI configuration files and scripts"
echo "${colok}chore    ${colreset}| Chores                   | Other changes that don't modify src or test files"
echo "${colok}revert   ${colreset}| Reverts                  | Reverts a previous commit"
echo ""
echo "Read the full specification here: https://www.conventionalcommits.org/en/v1.0.0/"
exit 1
```
