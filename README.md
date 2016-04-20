# Jupyter Stacks for OpenShift 3

This project provides the means of building Jupyter stack images for Docker which can run correctly in a secured Docker environment.

The Jupyter Docker images do already run as a non ``root`` user, however how that is done is not sufficient for a secured Docker environment which takes all steps to ensure you cannot run as ``root``, or somehow broach security of other running applications in a multi tenant hosting environment for Docker images.

The problems with the Jupyter images are as follows:

1. Jupyter images set ``USER`` to a named used rather than an integer UID. This means that the hosting service cannot actually properly verify that the user the container runs as, will not actually be running as ``root``. This is because an image could have added the named user but given it the ``root`` UID of ``0``. It would not be possible to detect this from looking at the Docker image meta data. In a secured Docker environment, ``USER`` of any images should always use an integer UID.
2. Jupyter images will not run if the hosting environment overrides the UID that the container run as to a value different to that specified by the ``USER``. This is because directories/files have ownership and permissions which prohibit the assigned user from reading or writing to them. To work in such an environment, all directories/files created should have ownership of the GID for the ``root`` group, this being the default GID used by Docker when containers are run. The ``HOME`` environment variable should also be set explicitly to deal with the case that the assigned UID when run does not have a corresponding UNIX account.

Although substitute images are provided here, this is seen as an interim measure. Ideally the original Jupyter images can be modified to work correctly.

## Changes Made

The Docker build files provided are a small shim on top of the official Jupyter stack images. All the ``Dockerfile`` does for each image is:

1. Set group ownership for directories/files under ``/home/jovyan`` to the ``root`` user.
2. Set group ownership for directories/files under ``/opt/conda`` to the ``root`` user.
3. Ensure that directories under ``/home/jovyan`` are accessible/writable to the ``root`` group.
4. Ensure that directories under ``/opt/conda`` are accessible/writable to the ``root`` group.
5. Ensure that files under ``/home/jovyan`` are readable/writable to the ``root`` group.
6. Ensure that files under ``/opt/conda`` are readable/writable to the ``root`` group.
7. Set the ``HOME`` environment variable to ``/home/jovyan``.

In addition to these fixes to the Jupyter stack images, the new derived image also sets image labels to enable the images to be used as [Source to Image](https://github.com/openshift/source-to-image) (S2I) builders. Such builders can be used in conjunction with the ``s2i`` tool to build Docker images which combine the base images with files from a Git repository, without a user need to know how to write ``Dockerfile`` themselves. This makes it very easy to bundle up a base image with a set of notebook into an image for distribution or deployment, such as in a teaching environment. The S2I function also works with OpenShift 3, enabling one click deployment of Jupyter images along with any required notebooks.

## Images Available

The list of Jupyter images for which fixed up variants are provided for are:

* [all-spark-notebook](https://github.com/jupyter/docker-stacks/tree/master/all-spark-notebook)
* [datascience-notebook](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook)
* [minimal-notebook](https://github.com/jupyter/docker-stacks/tree/master/minimal-notebook)
* [pyspark-notebook](https://github.com/jupyter/docker-stacks/tree/master/pyspark-notebook)
* [r-notebook](https://github.com/jupyter/docker-stacks/tree/master/r-notebook)
* [scipy-notebook](https://github.com/jupyter/docker-stacks/tree/master/scipy-notebook)

## Building the Images

Select which image you require for the work you need to do. Then run the OpenShift command line tool command ``oc new-build`` with this repository, the context directory set to the image name, and an output image name.

It is important to prefix the output image name with ``jupyter-`` else the name will clash with the Jupyter stack pulled down as the base image.

The commands to build each of the following would therefore be as follows:

**all-spark-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-all-spark-notebook --context-dir=all-spark-notebook
```

**datascience-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-datascience-notebook --context-dir=datascience-notebook
```

**minimal-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-minimal-notebook --context-dir=minimal-notebook
```

**pyspark-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-pyspark-notebook --context-dir=pyspark-notebook
```

**r-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-r-notebook --context-dir=r-notebook
```

**scipy-notebook**

```
oc new-build https://github.com/GrahamDumpleton/openshift3-jupyter-stacks.git --name jupyter-scipy-notebook --context-dir=scipy-notebook
```

## Using the Images

If you only need an empty Jupyter Notebook instance and will upload any notebooks manually, you can deploy the image directly. If for example needing the ``minimal-notebook``, you would run:

```
oc new-app jupyter-minimal-notebook --name my-notebook
oc expose service my-notebook
```

By default the service will be exposed via HTTP and will not be password protected. It is recommended you modify the route to enable TLS edge termination.

To enable a password, instead of the commands above, instead use:

```
oc new-app jupyter-minimal-notebook --name my-notebook --env PASSWORD=mypassword
oc expose service my-notebook
```

To have a set of notebooks and other data files combined with the image and deployed in one step, run the image as an S2I builder. That is, image name followed by ``~`` and the Git repository URL containing the notebooks.

```
oc new-app jupyter-minimal-notebook~https://github.com/jrjohansson/scientific-python-lectures.git --name my-notebook
oc expose service my-notebook
```

## Application Templates

As well as being able to deploy instances of the Jupyter Notebooks from the command line using the above commands, application templates are also provided. Once loaded these can be used from the OpenShift web console or the command line.

These templates will automatically provision a password and also enabled a secure HTTPS route for accessing the instance.

To load the application templates for the image you are interested in, use the appropriate command below.

**all-spark-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/all-spark-notebook.json
```

**datascience-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/datascience-notebook.json
```

**minimal-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/minimal-notebook.json
```

**pyspark-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/pyspark-notebook.json
```

**r-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/r-notebook.json
```

**scipy-notebook**

```
oc create -f https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/openshift-templates/scipy-notebook.json
```

## Using the Templates

To use the templates, once loaded, from the web console, use *Add to Project* within the project you want to use.

To use the templates from the command line, determine the names using ``oc get templates``.

```
$ oc get templates
NAME                       DESCRIPTION                   PARAMETERS    OBJECTS
jupyter-minimal-notebook   Jupyter (minimal-notebook).   4 (3 blank)   5
```

You can use the parameters using the ``oc describe`` command.

```
$ oc describe template jupyter-minimal-notebook
Name:		jupyter-minimal-notebook
Created:	15 minutes ago
Labels:		<none>
Description:	Jupyter (minimal-notebook).
Annotations:	iconClass=instant-app
		tags=instant-app

Parameters:
    Name:		APPLICATION_NAME
    Display Name:	Name
    Description:	Identifies the resources created for this application.
    Required:		true
    Value:		<none>
    Name:		REPOSITORY_URL
    Display Name:	Git Repository URL
    Description:	Repository for your Jupyter Notebooks and data.
    Required:		true
    Value:		<none>
    Name:		USER_PASSWORD
    Display Name:	User Password
    Description:	User password for accessing Jupyter Notebook.
    Required:		true
    Generated:		expression
    From:		[a-zA-Z0-9]{8}

    Name:		ROUTE_HOSTNAME
    Display Name:	Hostname
    Description:	Public hostname for the route. If not specified, a hostname is generated.
    Required:		false
    Value:		<none>

Object Labels:	<none>

Objects:
    ImageStream		${APPLICATION_NAME}
    BuildConfig		${APPLICATION_NAME}
    DeploymentConfig	${APPLICATION_NAME}
    Service		${APPLICATION_NAME}
    Route		${APPLICATION_NAME}
```

The ``oc new-app`` command can then be used to create and instance of the Jupyter Notebook for that image.

```
$ oc new-app jupyter-minimal-notebook-app --param APPLICATION_NAME=my-notebook --param REPOSITORY_URL=https://github.com/jrjohansson/scientific-python-lectures.git
--> Deploying template jupyter-minimal-notebook-app for "jupyter-minimal-notebook-app"
     With parameters:
      Name=my-notebook
      Git Repository URL=https://github.com/jrjohansson/scientific-python-lectures.git
      User Password=XkenopKc # generated
      Hostname=
--> Creating resources with label app=my-notebook ...
    imagestream "my-notebook" created
    buildconfig "my-notebook" created
    deploymentconfig "my-notebook" created
    service "my-notebook" created
    route "my-notebook" created
--> Success
    Build scheduled, use 'oc logs -f bc/my-notebook' to track its progress.
    Run 'oc status' to view your app.
```

When a password is not specified, you can see the auto generated password in the output of ``oc new-app``. In the web console you can see it in the environment variables associated with the deployment configuration.

## Using Persistent Volumes

By default any notebooks are not persistent. If the pod for the instance is destroyed, any work will be lost.

If you wish for data to be persistent, you will need to make a persistent volume claim and then mount the volume into the instance. The volume should be mounted on the directory ``/home/jovyvan/work``.

If you already have an empty Jupyter Notebook instance running, you can use the following command to claim a persistent volume and mount it.

```
oc set volume dc/my-notebook --add --name=my-notebooks -t pvc --claim-size=1G --overwrite --mount-path=/home/jovyan/work
```

