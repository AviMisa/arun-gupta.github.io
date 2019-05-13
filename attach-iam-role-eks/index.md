# Add IAM users to Amazon EKS Cluster

## Destination Machine

- Create an IAM role:

	```
	aws iam create-role \
		--role-name myeksrole \
		--assume-role-policy-document file://trust-policy.json
	```

  Shows the output:

  ```
	{
    "Role": {
      "Path": "/",
      "RoleName": "myeksrole",
      "RoleId": "AROARKOFJSCVYESJO2ZZT",
      "Arn": "arn:aws:iam::<account-id>:role/myeksrole",
      "CreateDate": "2019-05-13T15:00:16Z",
      "AssumeRolePolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "Service": "eks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
          }
        ]
      }
    }
	}
	```

  Note `Arn` of the role.
- Install aws-iam-authenticator:

	```
	brew install aws-iam-authenticator
	```

## Master Machine

- Write kube config:

	```
	eksctl utils write-kubeconfig --name myeks --kubeconfig ./kubeconfig
	```

- (NEEDED?) Get `aws-auth` ConfigMap:

	```
	kubectl describe configmap -n kube-system aws-auth
	```

- Replace `$Arn` from the destination machine in the script below. Add the IAM role to aws-auth ConfigMap for the EKS cluster:

	ROLE="    - rolearn: $Arn\n      username: eks\n      groups:\n        - system:masters"
	kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml
	kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"

