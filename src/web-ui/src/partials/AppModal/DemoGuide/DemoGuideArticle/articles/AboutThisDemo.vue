<template>
  <ArticleLayout>
    <template #heading>About this demo</template>

    <p>
      The
      <a href="https://github.com/aws-samples/retail-demo-store" target="_blank" rel="noreferrer noopener"
        >EdTech Demo</a
      >
      is an open source solution which integrates Amazon Personalize, Amazon Pinpoint and Amazon Lex in a retail web
      application demo and workshop platform. The EdTech Demo is intended to be used as an education tool (not
      designed for production usage) for AWS customers to demonstrate how AWS can be used to build compelling customer
      experiences for eCommerce, Retail and Digital Marketing use cases.
    </p>

    <p>
      The EdTech Demo is a fictitious online Course website that includes users (students), courses, carts and orders as
      well as services for search and recommendations and it helps demonstrate how the course recommendations change
      based on student's preferences, real-time behavior and history.
    </p>

    <p>
      Throughout the Ed Tech site experience, users are delivered (or have the option to be delivered) engagements across
      multiple channels including email and text (SMS). These messages occur at key moments in the student lifecycle,
      such as signup and abandoned cart. As users interact with the demo store and their emails/texts, real-time
      segments based on their behavior are created and then personalized engagements across channels are delivered. By
      utilizing Amazon Pinpoint and Amazon Personalize together, student interactions across each of these channels
      (store, email, SMS) are ingested as a single event stream and fed into the demo store personalization model,
      achieving omni-channel ML model retraining. On the other side, the ML personalization model is used to deliver
      personalized experiences across each of these channels a harmonized way.
    </p>

    <section class="my-4">
      <h2 class="section-heading">Architecture</h2>

      <p>
        The core of the EdTech Demo is a polyglot microservice architecture deployed as a collection of RESTful
        web services in Amazon Elastic Container Service (ECS). Several AWS managed services are leveraged to provide
        build, deployment, authentication, messaging, search, and personalization capabilities. The web user interface
        is a single page application built using responsive web design frameworks and techniques, producing a native
        app-like experience tailored to the user's device. See the
        <a
          href="https://github.com/aws-samples/retail-demo-store/tree/master/workshop"
          target="_blank"
          rel="noreferrer noopener"
          >workshops page</a
        >
        for details.
      </p>

      <div class="my-4"><img src="/rds-architecture.png" alt="EdTech Demo architecture" class="img-fluid" /></div>
    </section>

    <section class="my-4">
      <h2 class="section-heading">How the EdTech Demo works</h2>

      <p>
        The EdTech Demo is designed to display how personalization works and the impact it can have on a students
        experience. The demo has two main components: first, the storefront user interface and second, the demo guide
        which walks the user through all the different components of the web application demo. .
      </p>

      <p>
        The storefront user interface is the actual online store. The products presented will change based on the
        shopping preferences of the fictitious “student” profile selected. Users will be prompted to select a “student”
        after account creation or sign in and will be able to switch “student” by selecting the “student”details on the
        top-right corner. The demo guide is an educational component that sits on top of the store-front and is designed
        to guide users.
      </p>

      <p>
        The demo guide provides more information on how to use the demo during customer meetings. It also includes the
        services and use cases enabled in the demo, as well as some frequently asked Personalize topics.
      </p>
    </section>
  </ArticleLayout>
</template>

<script>
import ArticleLayout from '../ArticleLayout';

export default {
  name: 'AboutThisDemo',
  components: { ArticleLayout },
};
</script>

<style scoped>
.section-heading {
  font-weight: normal;
  font-size: 1.5rem;
}
</style>
