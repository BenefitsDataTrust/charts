COMMIT_SHA := $(shell git rev-parse --verify HEAD)

activate:
		echo If this fails, ensure you have created a $(PROJECT) gcloud configuration
		gcloud config configurations activate $(PROJECT)
		gcloud container clusters get-credentials cluster

activate.bdt-eng-public: PROJECT=bdt-eng-public
activate.bdt-eng-public: activate

wp.install:
		helm upgrade --install bdtrust-external --values=wp-values.yaml stable/wordpress

cluster-info:
		gcloud container clusters get-credentials cluster

helm.init: cluster-info
		kubectl create serviceaccount --namespace kube-system tiller
		kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
		sleep 5
		helm init --service-account tiller --upgrade
		sleep 5
		kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

deploy.staging.step-1: activate.bdt-eng-public
	helm upgrade --install staging .

deploy.staging.step-2:
	gsutil cp gs://bdt-public-assets/benefits-data-trust_live_2018-11-08T21-00-00_UTC_code.tar.gz .
	gsutil cp gs://bdt-public-assets/benefits-data-trust_live_2018-11-08T21-00-00_UTC_database.sql.gz .
	gsutil cp gs://bdt-public-assets/benefits-data-trust_live_2018-11-08T21-00-00_UTC_files.tar.gz .


