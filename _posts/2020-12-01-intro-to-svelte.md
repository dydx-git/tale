---
layout: post
title: "Svelte | My favourite JS framework for a great DX"
categories: Code
author: "Muhammad Taha"
---

I'm a sucker for underdogs. I love to give them a chance. When learning something new I'd much rather go for a better but lesser known framework. 
Of course, something only being off the beaten track shouldn't be the only attribute it should have. It should also be better than its more popular counterpart.

And svelte fits the bill perfectly. It's a lean and super fast JS framework that disappears once the code has been built. Yes, the apps developed using svelte are "built". Svelte has a compiler that traspiles `.svelte` files to vanilla javascript. 
Unlike React or Vue, Svelte is inherently reactive. Reactivity is built into the language and doesn't pollute your code with `useState` and such.

It's almost always a better idea to use some form validation framework but lets try to build one in Svelte just for the sake of learning it.

## Forms in Svelte:
```javascript
<script>
  import { onMount, onDestroy } from "svelte";
  import { getData } from "../../services/fetchData.js";
  import Spinner from "../Spinner.svelte";
  import Select from "../select/Select.svelte";
  import {
    employeesStore,
    payMethodsStore,
  } from "./stores/index2";

  export let submit;
  export let cancel = () => (showForm = false);

  export let showForm = false;
  export let title = "Form";

  export let clientName = "";
  export let email = "";
  export let phone = "";
  export let address = "";
  export let selectedPaymentMethod;
  export let selectedEmployee;

  let isMounted;
  let allPromises;

  $: employeesData = $employeesStore;
  $: payMethodData = $payMethodsStore;
</script>
```
`export`ing variables in the above few initial lines of code makes those variables available as props of this component to its parent. Going down further, you'll notice this weird syntax with `$:`. What this does is tell the JS engine the variable on the left depends on the one on the right side of `=` and that it should update the dependent variable whenever any changes occur. This should make more sense later on.

### Validation Logic
Here's the actual form validation logic:
```javascript
  export let isFormInvalid;

  let isClientNameDirty = false;
  let isEmailDirty = false;
  let isPhoneDirty = false;
  let isAddressDirty = false;
  let isPaymentMethodDirty = false;
  let isCompanyDirty = false;
  let isEmployeeDirty = false;

  $: isClientNameEmpty = clientName.length < 1 ? true : false;
  $: isEmailInvalid = !validateEmail(email.split(" ")[0]);
  $: isPhoneEmpty = phone.length < 1 ? true : false;
  $: isAddressEmpty = address.length < 1 ? true : false;

  $: isPaymentMethodEmpty =
    isPaymentMethodDirty && selectedPaymentMethod == null ? true : false;

  $: isEmployeeEmpty =
    isEmployeeDirty && selectedEmployee == null ? true : false;

  $: isFormInvalid =
    isClientNameEmpty ||
    isPhoneEmpty ||
    isEmailInvalid ||
    isAddressEmpty ||
    isPaymentMethodEmpty ||
    isCompanyEmpty ||
    isEmployeeEmpty;

  function validateEmail(value) {
    return (
      (value &&
        !!value.match(
          /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
        )) ||
      false
    );
  }
```

Lets focus on the `isClientNameEmpty` line and see how that works. Later on in the HTML of this form, you'll see that `clientName` is binding to a textbox. Whenever the user enters any text in the textbox, the `isClientNameEmpty` line will reevaluate whatever's on the right side and update the dependent variable. So, each letter entered will trigger a reaction and in turn will update the `isClientNameEmpty` variable. 

A point to be noted here: if there's multiple variables being evaluated then the update will trigger if any of them change. So, for example, `isPaymentMethodEmpty` line will reevaluate whenever either `isPaymentMethodDirty` or `selectedPaymentMethod` gets changed.
