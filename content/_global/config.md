+++
fragment = "config"

# The weight doesn't matter as this fragment doesn't show something but only loads resources.
weight = 0

# Configure cookie banner: load css and js of Eclipse's cookie banner
[[config]]
type = "css"
resource = "https://www.eclipse.org/eclipse.org-common/themes/solstice/public/stylesheets/vendor/cookieconsent/cookieconsent.min.css"
[[config]]
type = "js"
resource = "https://www.eclipse.org/eclipse.org-common/themes/solstice/public/javascript/vendor/cookieconsent/default.min.js"
+++