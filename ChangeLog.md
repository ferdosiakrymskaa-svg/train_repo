export GITHUB_USERNAME=ferdosiakrymskaa-svg
export GITHUB_EMAIL=ferdosiakrumskaa@gmail.com
export DATE="`LANG=en_US date +'%a %b %d %Y'`"
cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
