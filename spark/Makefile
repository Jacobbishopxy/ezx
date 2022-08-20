# Test case

install:
	helm install -n dev -f my-values.yaml xy-spark .

upgrade:
	helm upgrade -n dev -f my-values.yaml xy-spark .

uninstall:
	helm uninstall -n dev xy-spark

node-check:
	kubectl get nodes --show-labels

node-create:
	kubectl label node k8s-node1 xy-spark=right
	kubectl label node k8s-node2 xy-spark=right
	kubectl label node k8s-node3 xy-spark=right
	kubectl label node k8s-node4 xy-spark=right

node-delete:
	kubectl label node k8s-node1 xy-spark-
	kubectl label node k8s-node2 xy-spark-
	kubectl label node k8s-node3 xy-spark-
	kubectl label node k8s-node4 xy-spark-

