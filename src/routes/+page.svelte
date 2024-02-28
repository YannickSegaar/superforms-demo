<script lang="ts">
    import { superForm } from "sveltekit-superforms/client"
    import SuperDebug from "sveltekit-superforms/client/SuperDebug.svelte"
    import { z } from "zod"
    import type { PageData } from "./$types"
    const airtableApiKey = import.meta.env.VITE_AIRTABLE_API_KEY;


    export let data: PageData

    const newContactSchema = z.object({
        firstName: z.string().min(1),
        lastName: z.string().min(1),
        email: z.string().email(),
        telefoon: z.string().min(1)
    })

    const { form, errors, enhance, constraints } = superForm(data.form, {
        taintedMessage: "Are you sure you want leave?",
        validators: newContactSchema,
        SPA: true, // Ensure SPA mode is enabled for onUpdate functionality
        onUpdate: async ({ form }) => {
            if (form.valid) {
                const airtableData = {
                    records: [
                        {
                            fields: {
                                "First Name": form.data.firstName,
                                "Last Name": form.data.lastName,
                                "Email": form.data.email,
                                "Telefoon": form.data.telefoon,
                            }
                        }
                    ]
                };

                const response = await fetch('https://api.airtable.com/v0/appiUhN3UTNOzb1v3/Svelte%20Contactform', {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${airtableApiKey}`,
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(airtableData)
                });

                if (response.ok) {
                    // Handle success - maybe navigate to a thank you page or show a success message
                    console.log('Submission successful');
                } else {
                    // Handle error - maybe show an error message to the user
                    console.error('Submission failed');
                }
            }
        }
    });
</script>

<SuperDebug data={$form} />


<article>
    <header>
        <h1>Contactgegevens</h1>
    </header>
    <form method="POST" use:enhance>
        <label for="firstName">First name</label>
        <input type="text" id="firstName" name="firstName" bind:value={$form.firstName} />
        {#if $errors.firstName}
            <small>{$errors.firstName}</small>
        {/if}
        <label for="lastName">Last name</label>
        <input type="text" id="lastName" name="lastName" bind:value={$form.lastName} placeholder="1234 AA, 12" />
        {#if $errors.lastName}
            <small>{$errors.lastName}</small>
        {/if}
        <label for="email">Email address</label>
        <input type="email" id="email" name="email" bind:value={$form.email} />
        {#if $errors.email}
            <small>{$errors.email}</small>
        {/if}
        <label for="telefoon">Telefoonnummer</label>
        <input type="text" id="telefoon" name="telefoon" bind:value={$form.telefoon} />
        {#if $errors.telefoon}
            <small>{$errors.telefoon}</small>
        {/if}
        <button type="submit">Submit</button>
    </form>
</article>