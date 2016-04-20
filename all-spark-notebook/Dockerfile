FROM jupyter/all-spark-notebook

USER root

RUN chgrp -R root /home/$NB_USER \
    && find /home/$NB_USER -type d -exec chmod g+rws {} \; \
    && find /home/$NB_USER -type f -exec chmod g+rw {} \;

ENV HOME /home/$NB_USER

USER 1001

LABEL io.k8s.description="S2I builder for Jupyter (all-spark-notebook)." \
      io.k8s.display-name="Jupyter (all-spark-notebook)" \
      io.openshift.expose-services="8888:http" \
      io.openshift.tags="builder,python,scala,r,spark,jupyter" \
      io.openshift.s2i.scripts-url="https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/s2i-bootstrap"
