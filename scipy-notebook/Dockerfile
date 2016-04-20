FROM jupyter/scipy-notebook

USER root

RUN chgrp -R root /home/$NB_USER && chmod -Rf g+w /home/$NB_USER

ENV HOME /home/$NB_USER

USER 1001

LABEL io.k8s.description="S2I builder Jupyter (scipy-notebook)." \
      io.k8s.display-name="Jupyter (scipy-notebook)" \
      io.openshift.expose-services="8888:http" \
      io.openshift.tags="builder,python,jupyter" \
      io.openshift.s2i.scripts-url="https://raw.githubusercontent.com/GrahamDumpleton/openshift3-jupyter-stacks/master/s2i-bootstrap"
