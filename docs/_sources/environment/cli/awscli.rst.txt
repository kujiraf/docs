AWS CLI
========================================

Install
----------------------------------------
asdfでバージョン管理を行う。

.. code-block:: bash

   # add plugin
   asdf plugin add awscli
   # show plugin list
   asdf plugin list
   # install
   asdf install awscli latest
   # show install list
   asdf list
   # set primary version
   asdf global awscli 2.8.6

Profileの変更
----------------------------------------
.. code-block:: bash

   # change profile
   export AWS_PROFILE=[~/.aws/credentials上のprofile]
   # show identity
   aws sts get-caller-identity
