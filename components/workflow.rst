.. index::
   single: Workflow
   single: Components; Workflow

The Workflow Component
======================

    The Workflow component provides tools for managing a workflow or finite
    state machine.

Installation
------------

.. code-block:: terminal

    $ composer require symfony/workflow

.. include:: /components/require_autoload.rst.inc

Creating a Workflow
-------------------

The workflow component gives you an object oriented way to define a process
or a life cycle that your object goes through. Each step or stage in the
process is called a *place*. You do also define *transitions* that describe
the action to get from one place to another.

.. image:: /_images/components/workflow/states_transitions.png

A set of places and transitions creates a **definition**. A workflow needs
a ``Definition`` and a way to write the states to the objects (i.e. an
instance of a :class:`Symfony\\Component\\Workflow\\MarkingStore\\MarkingStoreInterface`).

Consider the following example for a blog post. A post can have one of a number
of predefined statuses (`draft`, `review`, `rejected`, `published`). In a workflow,
these statuses are called **places**. You can define the workflow like this::

    use Symfony\Component\Workflow\DefinitionBuilder;
    use Symfony\Component\Workflow\MarkingStore\SingleStateMarkingStore;
    use Symfony\Component\Workflow\Transition;
    use Symfony\Component\Workflow\Workflow;

    $definitionBuilder = new DefinitionBuilder();
    $definition = $definitionBuilder->addPlaces(['draft', 'review', 'rejected', 'published'])
        // Transitions are defined with a unique name, an origin place and a destination place
        ->addTransition(new Transition('to_review', 'draft', 'review'))
        ->addTransition(new Transition('publish', 'review', 'published'))
        ->addTransition(new Transition('reject', 'review', 'rejected'))
        ->build()
    ;

    $marking = new SingleStateMarkingStore('currentState');
    $workflow = new Workflow($definition, $marking);

The ``Workflow`` can now help you to decide what actions are allowed
on a blog post depending on what *place* it is in. This will keep your domain
logic in one place and not spread all over your application.

When you define multiple workflows you should consider using a ``Registry``,
which is an object that stores and provides access to different workflows.
A registry will also help you to decide if a workflow supports the object you
are trying to use it with::

    use Acme\Entity\BlogPost;
    use Acme\Entity\Newsletter;
    use Symfony\Component\Workflow\Registry;
    use Symfony\Component\Workflow\SupportStrategy\InstanceOfSupportStrategy;

    $blogWorkflow = ...
    $newsletterWorkflow = ...

    $registry = new Registry();
    $registry->addWorkflow($blogWorkflow, new InstanceOfSupportStrategy(BlogPost::class));
    $registry->addWorkflow($newsletterWorkflow, new InstanceOfSupportStrategy(Newsletter::class));

Usage
-----

When you have configured a ``Registry`` with your workflows, you may use it as follows::

    // ...
    $post = new BlogPost();
    $workflow = $registry->get($post);

    $workflow->can($post, 'publish'); // False
    $workflow->can($post, 'to_review'); // True

    $workflow->apply($post, 'to_review');
    $workflow->can($post, 'publish'); // True
    $workflow->getEnabledTransitions($post); // ['publish', 'reject']

Learn more
----------

.. toctree::
    :maxdepth: 1
    :glob:

    /workflow/*

.. _Packagist: https://packagist.org/packages/symfony/workflow
