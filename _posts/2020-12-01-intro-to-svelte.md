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

Form Validation:
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
  export let selectedCompany;
  export let selectedEmployee;

  let isMounted;
  let allPromises;

  $: employeesData = $employeesStore;
  $: payMethodData = $payMethodsStore;

  // form validation
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

  $: isCompanyEmpty = isCompanyDirty && selectedCompany == null ? true : false;

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
  // end form validation

  const fetchData = () => {
    if (
      isMounted &&
      $employeesStore.length === 0 &&
      $payMethodsStore.length === 0
    ) {
      let fetchEmployees = [
        {value: 1, label: 'Employee 1'}, 
        {value: 2, label: 'Employee 2'}, 
        {value: 3, label: 'Employee 3'}, 
      ];
      let fetchPayMethods = [
        {value: 1, label: 'PayPal'}, 
        {value: 2, label: 'Credit Card'}, 
        {value: 3, label: 'Check'}, 
      ];

      allPromises = Promise.all([
        fetchEmployees,
        fetchPayMethods,
      ]);
      (async function () {
        $employeesStore = await fetchEmployees;
        $payMethodsStore = await fetchPayMethods;
      })();
    }
  };

  const handleKeydown = (e) => {
    if (e.key === "Escape") {
      cancel();
    }
  };

  onMount(() => {
    isMounted = true;
    fetchData();
  });

  onDestroy(() => {
    isMounted = false;
  });
</script>

<svelte:window on:keydown="{handleKeydown}" />

{#await allPromises}
  <Spinner />
{/await}

<form class="container mx-auto my-48 w-full max-w-lg">
  <div>
    <h2 class="text-2xl my-6 font-semibold leading-tight">{title}</h2>
  </div>
  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full md:w-2/5 px-3 mb-6 md:mb-0">
      <input
        bind:value="{clientName}"
        class="appearance-none block w-full text-gray-900 border
        border-black-900 rounded py-3 px-4 leading-tight focus:outline-none
        focus:bg-white focus:border-blue-600"
        type="text"
        placeholder="Client Name"
        on:focus="{() => (isClientNameDirty = true)}"
        autofocus />
      {#if isClientNameDirty && isClientNameEmpty}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
    <div class="w-full md:w-3/5 px-3 mb-6 md:mb-0">
      <input
        bind:value="{email}"
        class="appearance-none block text-gray-900 border border-black-900
        rounded py-3 px-4 leading-tight focus:outline-none focus:bg-white
        focus:border-gray-500 w-full"
        type="text"
        placeholder="Email Address"
        on:focus="{() => (isEmailDirty = true)}" />
      {#if isEmailDirty && isEmailInvalid}
        <p class="text-red-500 text-xs italic">
          Please enter a correct email address.
        </p>
      {/if}
    </div>
  </div>

  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full md:w-3/5 px-3 mb-6 md:mb-0">
      <input
        bind:value="{phone}"
        class="appearance-none block w-full text-gray-900 border
        border-black-900 rounded py-3 px-4 leading-tight focus:outline-none
        focus:bg-white focus:border-blue-600"
        type="text"
        placeholder="(XXX) XXX-XXXX"
        on:focus="{() => (isPhoneDirty = true)}" />
      {#if isPhoneDirty && isPhoneEmpty}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
    <div class="w-full md:w-2/5 px-3 mb-6 md:mb-0">
      <Select
        items="{payMethodData}"
        bind:selectedItemId="{selectedPaymentMethod}"
        on:focus="{() => (isPaymentMethodDirty = true)}"
        placeholder="Select Payment Method" />
      {#if isPaymentMethodDirty && isPaymentMethodEmpty}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
  </div>

  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full px-3 mb-6 md:mb-0">
      <input
        bind:value="{address}"
        class="appearance-none block w-full text-gray-900 border
        border-black-900 rounded py-3 px-4 leading-tight focus:outline-none
        focus:bg-white focus:border-blue-600"
        type="text"
        placeholder="Street Address, City, Zipcode, State"
        on:focus="{() => (isAddressDirty = true)}" />
      {#if isAddressDirty && isAddressEmpty}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
  </div>

  <div class="flex flex-wrap -mx-3 mb-6">
    <div class="w-full md:w-1/2 px-3 mb-6 md:mb-0">
      <Select
        items="{employeesData}"
        bind:selectedItemId="{selectedEmployee}"
        on:focus="{() => (isEmployeeDirty = true)}"
        placeholder="Select Employee" />
      {#if isEmployeeDirty && isEmployeeEmpty}
        <p class="text-red-500 text-xs italic">Please fill out this field.</p>
      {/if}
    </div>
  </div>

  <div
    class="px-5 py-5 bg-white border-t flex flex-col xs:flex-row items-center
    xs:justify-between ">
    <div class="mt-2 px-2 xs:mt-0 items-center">
      <button
        class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold
        py-2 px-4 rounded-l shadow"
        on:click="{submit}"
        type="button">
        {title.split(' ')[0]}
      </button>
      <button
        class="text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 font-semibold
        py-2 px-4 rounded-r shadow"
        on:click="{cancel}"
        type="button">
        Cancel
      </button>
    </div>
  </div>
</form>
```
