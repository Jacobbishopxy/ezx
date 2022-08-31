# Test case

install:
	helm install -n dev -f my-values.yaml xy-pg .

install-prod:
	helm install -n prod -f prod-values.yaml xy-prod-pg .

uninstall:
	helm uninstall -n dev xy-pg

uninstall-prod:
	helm uninstall -n prod xy-prod-pg
