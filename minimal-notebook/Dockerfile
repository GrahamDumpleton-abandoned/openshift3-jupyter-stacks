FROM jupyter/minimal-notebook

USER root

RUN chgrp -R root /home/$NB_USER && chmod -Rf g+w /home/$NB_USER

ENV HOME /home/$NB_USER

USER 1001
