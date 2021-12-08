# Kubernetes "Demo": Intro to Helm (conclusion); Intro to RBAC

## Goals

* Conclude our helm survey
* Survey Kubernetes RBAC (role-based access control)
  * for users, specifically in context of an AWS EKS cluster in our demo example
  * for pods running in the cluster

## Concepts

* RBAC
  * Role
  * ClusterRole
  * RoleBinding
  * ClusterRoleBinding
  * Subjects
    * User
    * ServiceAccount
  * K8s API server authentication options

## Lab

### 1. Setup

* Follow the updated README in the `todoapi` [here](https://github.com/itt-learning-groups/todoapi/blob/master/build/README.md) to get the demo cluster up and running.

### 2. Helm Survey (concluded)

### 3. RBAC I: Setting up IAM-based admin & developer access roles for the cluster

* Create IAM users representing our example admin ("ImanAdmin") and developer ("ImaDeveloper") users.

!["001"](img/001.png "001")

!["002"](img/002.png "002")

#### Admin user(s)/role/group

* Create an IAM user group for K8s admin users; add "ImanAdmin" to it.

!["003"](img/003.png "003")

* Create an IAM role for all users (e.g. "ImanAdmin") who will have admin access to the cluster.
  Remember: K8s doesn't manage users. You need to manage your users and user groups outside K8s.
  There are many options. Since we're already using AWS for our (EKS) example cluster here, we'll use AWS IAM to manage our users for this particular example, since that's convenient and secure.

  * We want the IAM role to be "assumeable" by any IAM users in the "learnanddevopsClusterAdmin" group.
    We'll make an assume-role IAM policy for the role that allows users within the group to assume the role.
    (IAM doesn't allow a user group to be assigned as a principal here; we'll need to ammend the principal users when/if new users are added to the cluster admin group in the future.)

    !["004"](img/004.png "004")

  * Then we'll add a policy to the "learnanddevopsClusterAdmin" group allowing its members specific permission to assume the role.

    !["005"](img/005.png "005")

* Register the "learnanddevopsClusterAdmin" IAM role with our EKS cluster's aws-iam-authenticator plugin

  * In order to authenticate externally-managed users with the K8s cluster's API server, your cluster must be configured with an authenticator.
    Here again, there are a number of alternatives.
    In our example, our EKS cluster is already configured with the [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator#how-does-it-work), courtesy of our easy `eksctl` cluster setup.
    This authenticator is already running on the cluster's control plane, and it maps IAM roles or users to K8s-internal users via a ConfigMap called `aws-auth` in the `kube-system` namespace.
    There is only one role mapped there so far, which was done by eksctl to enable the NodeInstanceRole that is managing the cluster's EC2 node instances:

    !["006"](img/006.png "006")

    We could edit this ConfigMap manually, but it's convenient to use `eksctl` for that.
    For this "admin" role, we don't need to create a new K8s Role/ClusterRole and RoleBinding/ClusterRoleBinding.
    Our cluster already has a convenient `system:masters` group bound to the existing `cluser-admin` ClusterRole. We'll re-use that in this case:

    !["011"](img/011.png "011")

    !["012"](img/012.png "012")

    !["007"](img/007.png "007")

    !["008"](img/008.png "008")

* Create a local AWS profile to capture that new learnanddevopsClusterAdmin IAM role so we can assume it and try it out (as if we were the "ImanAdmin" user):

    !["009"](img/009.png "009")

* Update `~/.kube/config` to extend kubectl access for this profile:

    !["010"](img/010.png "010")

* Assume the AWS profile and use it to manage the cluster, e.g.:

    !["013"](img/013.png "013")

#### Developer user(s)/role/group

A lot of this will be the same as for the "Admin" user/role/group.
But instead of using an existing ClusterRole and binding like we did for the "learnanddevopsClusterAdmin", we'll create custom Role and RoleBinding within the `dev` and `qa` namespaces with non-admin permissions.
It would be a good idea to create a 3rd set of user(s)/role/group for more limited, principally read-only access within the `prod` namespace. But we'll leave that as a homework exercise.
The goal is to offer a limited, non-admin set of permissions for ordinary developers to use: Allow them to do their jobs but limit the amount of damage they might be able to do through ignorance, carelessness, or malice.

* Create an IAM user group for K8s developer users; add "ImaDeveloper" to it.

!["014"](img/014.png "014")

* Create an IAM role for all users (e.g. "ImaDeveloper") who will have developer access to the cluster.

  * We want the IAM role to be "assumeable" by any IAM users in the "learnanddevopsDeveloper" group.
    We'll make an assume-role IAM policy for the role that allows users within the group to assume the role.

    !["015"](img/015.png "015")

  * Then we'll add a policy to the "learnanddevopsDeveloper" group allowing its members specific permission to assume the role.

    !["016"](img/016.png "016")

* Create a K8s Role and RoleBinding

    !["017"](img/017.png "017")

    !["018"](img/018.png "018")

* Register the "learnanddevopsDeveloper" IAM role with our EKS cluster's aws-iam-authenticator plugin

    !["019"](img/019.png "019")

    !["020"](img/020.png "020")

* Create a local AWS profile to capture that new learnanddevopsDeveloper IAM role so we can assume it and try it out (as if we were the "ImaDeveloper" user):

    !["021"](img/021.png "021")

* Update `~/.kube/config` again to extend kubectl access for this profile:

    !["022"](img/022.png "022")

* Assume the AWS profile and use for limited cluster management *within the `dev` or `qa` namespaces*, but not in `prod` (not allowed for this kubectl user), e.g.:

    !["023"](img/023.png "023")

### 4. RBAC II: Setting up ServiceAccount-based role for `todoapi` pods in the cluster

(*Next Week*)
