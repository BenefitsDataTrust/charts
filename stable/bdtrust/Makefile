ARCHIVE_STRING := benefits-data-trust_live_2018-11-08T21-00-00_UTC_
POD := $(shell kubectl get po --output name | sed -e 's/.*\///' | grep -E "staging-bdtrust-[[:alnum:]]{10}-[[:alnum:]]{5}")

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
	gsutil cp gs://bdt-public-assets/$(ARCHIVE_STRING)code.tar.gz  ~/Desktop/
	gsutil cp gs://bdt-public-assets/$(ARCHIVE_STRING)database.sql.gz  ~/Desktop/
	# gsutil cp gs://bdt-public-assets/$(ARCHIVE_STRING)files.tar.gz  ~/Desktop/ # this file is huge

deploy.staging.step-3:
	kubectl cp ~/Desktop/$(ARCHIVE_STRING)code.tar.gz $(POD):/$(ARCHIVE_STRING)code.tar.gz
	kubectl exec $(POD) tar xzvf $(ARCHIVE_STRING)code.tar.gz
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/index.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/license.txt /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/readme.html /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-activate.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-admin /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-blog-header.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-comments-post.php /bitnami/wordpress
	kubectl exec $(POD) -- rm -rf /bitnami/wordpress/wp-content
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-content /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-cron.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-includes /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-links-opml.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-load.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-login.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-mail.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-settings.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-signup.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/wp-trackback.php /bitnami/wordpress
	kubectl exec $(POD) -- mv -f /$(ARCHIVE_STRING)code/xmlrpc.php /bitnami/wordpress

deploy.staging.step-4:
	kubectl cp ~/Desktop/benefits-data-trust_live_2018-11-08T21-00-00_UTC_database.sql.gz $(POD):/benefits-data-trust_live_2018-11-08T21-00-00_UTC_database.sql.gz
	kubectl exec $(POD) -- gzip -d /benefits-data-trust_live_2018-11-08T21-00-00_UTC_database.sql.gz
	sleep 2
	kubectl exec $(POD) -- bash -c "mysql -ubn_wordpress -p7FWRF457P395SDCPL29H56C8 -hstaging-mariadb bitnami_wordpress < /benefits-data-trust_live_2018-11-08T21-00-00_UTC_database.sql"
