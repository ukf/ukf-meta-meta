#
# Dockerfile to create an image that allows us to run an older
# (pre-2.13) version of Git to access the old filter-branch
# behaviour
#
FROM centos:7.4.1708

MAINTAINER Ian Young <ian@iay.org.uk>

#
# Install the version of Git native to this OS version, which is a
# variant of Git 1.8 with some backports.
#
RUN yum -y install git && \
    yum clean all

#
# Do all our work as a normal user, in a home directory.
#
RUN adduser user
USER user
ENV userdir=/home/user
WORKDIR ${userdir}

#
# By default, run a shell.
#
CMD ["/bin/bash"]
