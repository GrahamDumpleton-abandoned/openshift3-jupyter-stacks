FROM jupyter/pyspark-notebook

USER root

RUN chgrp -R root /home/$NB_USER && chmod -Rf g+w /home/$NB_USER

ENV HOME /home/$NB_USER

USER 1001

LABEL io.k8s.description="S2I builder for Jupyter (pyspark-notebook)." \
      io.k8s.display-name="Jupyter (pyspark-notebook)" \
      io.openshift.expose-services="8888:http" \
      io.openshift.tags="builder,python,spark,jupyter" \
      io.openshift.s2i.scripts-url="https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/s2i-bootstrap"
