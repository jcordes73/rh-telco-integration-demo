# Install SugarCRM on OpenShift

export SUGARCRM_PROJECT=sugarcrm
export SUGARCRM_RELEASE=demo

oc new-project $SUGARCRM_PROJECT
oc adm policy add-scc-to-user privileged -z default
oc adm policy add-scc-to-user privileged -z $SUGARCRM_PROJECT-mariadb

helm repo add bitnami https://charts.bitnami.com/bitnami
helm install $SUGARCRM_RELEASE bitnami/suitecrm

export APP_HOST=$(oc get route $SUGARCRM_RELEASE-suitecrm -o jsonpath="{.spec.host}")
export APP_PASSWORD=$(oc get secret --namespace $SUGARCRM_PROJECT $SUGARCRM_RELEASE-suitecrm -o jsonpath="{.data.suitecrm-password}" | base64 --decode)
export DATABASE_ROOT_PASSWORD=$(oc get secret --namespace $SUGARCRM_PROJECT $SUGARCRM_RELEASE-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 --decode)
export APP_DATABASE_PASSWORD=$(oc get secret --namespace $SUGARCRM_PROJECT $SUGARCRM_RELEASE-mariadb -o jsonpath="{.data.mariadb-password}" | base64 --decode)

helm upgrade $SUGARCRM_RELEASE bitnami/suitecrm \
    --set suitecrmHost=$APP_HOST,suitecrmPassword=$APP_PASSWORD,mariadb.auth.rootPassword=$DATABASE_ROOT_PASSWORD,mariadb.auth.password=$APP_DATABASE_PASSWORD

SUITECRM_PASSWORD="`oc get secret --namespace $SUGARCRM_PROJECT $SUGARCRM_RELEASE -o jsonpath="{.data.suitecrm-password}" | base64 --decode`"

echo -n $SUITECRM_PASSWORD | md5sum
