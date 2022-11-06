Terraform
========================================

`terraformとは <https://www.terraform.io/>`_
---------------------------------------------
.. todo:: 軽くまとめる

Install
----------------------------------------
asdfでバージョン管理を行う。

.. code-block:: bash

   # add plugin
   asdf plugin add terraform
   # show plugin list
   asdf plugin list
   # install
   asdf install terraform latest
   # show install list
   asdf list
   # set primary version
   asdf global terraform 1.3.3

[Tips] moduleにおけるresourceの競合するフィールドのON/OFF
----------------------------------------------------------

module側にlookup関数を使うことで実行可能

.. code-block:: javascript

   resource "aws_security_group_rule" "this" {
      security_group_id        = aws_security_group.this.id
      for_each                 = var.rules
      type                     = each.value[0]
      protocol                 = each.value[1]
      from_port                = each.value[2]
      to_port                  = each.value[3]
      description              = each.value[4]
      cidr_blocks              = lookup(each.value[5], "cidr_blocks", null) // 競合するオプション
      source_security_group_id = lookup(each.value[5], "source_sg", null)   // 競合するオプション
      self                     = lookup(each.value[5], "self", null)        // 競合するオプション
   }

呼び出し元はこのようにする

.. code-block:: javascript

   module "eks-cluster-sg" {
      source      = "../modules/sg"
      vpc_id      = data.aws_vpc.ma-personal-vpc.id
      name        = "${local.p}-eks-cluster-sg"
      description = "${local.p}-eks-cluster-sg"
      rules = {
         "ingress_all_self" = ["ingress", "tcp", 0, 65535, null, { self = true }] // lookupのため5番目の要素をmapにしている
         "egress_any"       = local.egress_any
      }
   }


[Tips] terraform consoleでの関数の実行結果
--------------------------------------------

forループ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   > [for d in toset([{a="a"},{a="b"},{a="c"}]) : lookup(d,"a")]
   [
      "a",
      "b",
      "c",
   ]

.. code-block:: javascript

   > [for d in toset([{a="a"},{a="b"},{a="c"}]) : d.a]
   [
      "a",
      "b",
      "c",
   ]

.. code-block:: javascript

   > [for d in [{a={key="aa"}},{b={key="bb"}}] : d]
   [
      {
         "a" = {
            "key" = "aa"
         }
      },
      {
         "b" = {
            "key" = "bb"
         }
      },
   ]

merge 複数のmapを1つのmapにまとめる
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   > merge([{a={key="aa"}},{b={key="bb"}}]...)
   {
      "a" = {
         "key" = "aa"
      }
      "b" = {
         "key" = "bb"
      }
   }

mergeとforを組み合わせて、複数のmapにおける特定の値を取得
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: javascript

   > [for i in merge([{a={key="aa"}},{b={key="bb"}}]...) : i.key]
   [
      "aa",
      "bb",
   ]
