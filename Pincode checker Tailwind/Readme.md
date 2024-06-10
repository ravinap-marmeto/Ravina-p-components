# Pincode Checker component

**Preview Link**
[here](https://ravina-test.myshopify.com/products/jodhpur-scalloped-hoops?preview_theme_id=151750246718)

Password - "cheaze"

## Overview

Integrate a Pincode-checker into your Shopify theme. This is built using Tailwind CSS, Javacsript and requires a google-sheet.

## Steps

1. **Add the Snippet:**

    Create a snippet named `pincode-checker.liquid` and paste the following code:
    ```html
    <script src="{{ 'pincode-checker.js' | asset_url }}" defer="defer"></script>

    <h4 class="heading">{{ block.settings.heading }}</h4>

    <pincode-checker class="pincode-checker-wrapper">
    <div class="field flex">
        <input
        id="pincode-{{ block.id }}"
        type="text"
        maxlength="6" 
        pattern="^\d{6}$"
        title="Please enter a valid 6-digit number"
        class="field__input"
        placeholder="Enter Pincode">
        <label for="pincode-{{ block.id }}" class="field__label">{{ block.settings.placeholder_text }}</label>
        <button class="button">{{ block.settings.button_text }}</button>
    </div>
    <p class="hidden success text-green-500 m-0" data-pincode-success>{{ block.settings.pincode_available_message }}</p>
    <p class="hidden failure text-red-500 m-0" data-pincode-failure>{{ block.settings.pincode_unavailable_message }}</p>
    <p class="hidden success text-green-500 m-0" data-pincode-estdelivery>Estimated Delivery Time: [est_Days]</p>
    </pincode-checker>
    ```

3. Create a asset file "pincode-checker.js" and paste the following code
```js
class PincodeChecker extends HTMLElement {
    constructor() {
        super();
        this.button = this.querySelector('button');
        this.input = this.querySelector('input');
        this.failureNode = this.querySelector('[data-pincode-failure]')
        this.successNode = this.querySelector('[data-pincode-success]')
        this.estDeliveryNode = this.querySelector('[data-pincode-estdelivery]')

        // Disabling the ATC and Buy now button by default.
         this.disableAddButton(true)

        // Check if data is already available in local storage, if yes, then use it.
        // Otherwise fetch the data from google sheets.
        this.data = JSON.parse(localStorage.getItem('pincodeSheetData')) || this.fetchSheetsData();

        this.reuseLocalStorageData();
        this.button.addEventListener('click', () => this.checkPincodeAvailability());
        this.input.addEventListener('keypress', function(e) {
            const char = e.key;
            if (!/^\d$/.test(char) || e.target.value.length === 6) {
                e.preventDefault();
            }
        });
    }

    async checkPincodeAvailability() {
        const pincode = this.input.value.trim();
        this.querySelectorAll('p').forEach(elem => elem.classList.add('hidden'))

        // Validate the pincode
        if (!pincode || pincode.length !== 6 || isNaN(pincode)) {
            alert('Please enter a valid 6-digit pincode');
            return;
        }

        try {
            const [header, result] = await this.filterDataFromGoogleSheet(pincode);

            if (result) {
                const deliverableIndex = header.indexOf('Deliverable');
                const EstDeliveryNode = this.querySelector('[data-pincode-estdelivery]')

                if (result[deliverableIndex] === 'TRUE') {
                    this.successNode.classList.remove('hidden');

                    const estDaysIndex = header.indexOf('Est Days');
                    this.estDeliveryNode.textContent = EstDeliveryNode.textContent.replace('[est_Days]', result[estDaysIndex])
                    this.estDeliveryNode.classList.remove('hidden');
                    
                    localStorage.setItem('pincode', pincode);

                    // Enabling the ATC and Buy now button.
                    this.disableAddButton(false)
                } else {
                    this.failureNode.classList.remove('hidden');
                    this.disableAddButton(true)
                }

            } else {
                this.failureNode.classList.remove('hidden');
                console.error('Pincode not found on sheet')
            }
        } catch (error) {
            console.error(error);
        }
    }

    async filterDataFromGoogleSheet(pincode) {
        this.data = await this.fetchSheetsData();
        
        const rows = this.data.values;
        const [header, ...dataRows] = rows;
        const result = dataRows.find(row => row[0] === pincode);

        return [header, result];
    }

    fetchSheetsData() {
        if (this.data) return this.data;

        const API_KEY = 'AIzaSyBBoDc5yP9MNwm3IrJBKd_71m_kqbOlSJE';
        const SPREADSHEET_ID = '1uXPz0o9CgxNFde3zBPJyXJ9gVpsIV58Y6VsAvygPnHM';
        const RANGE = 'Sheet1';
        const BASE_URL = `https://sheets.googleapis.com/v4/spreadsheets/${SPREADSHEET_ID}/values/${RANGE}`;
        const queryString = `?key=${API_KEY}&majorDimension=ROWS&valueRenderOption=FORMATTED_VALUE&dateTimeRenderOption=FORMATTED_STRING`;

        const url = `${BASE_URL}${queryString}`;
        return fetch(url)
            .then(response => response.json())
            .then(data => {
                // Store the data in local storage
                localStorage.setItem('pincodeSheetData', JSON.stringify(data));
                return data;
            })
            .catch(error => console.error(error));
    }

    reuseLocalStorageData() {
        if (!localStorage.getItem('pincode')) return; 

        this.input.value = localStorage.getItem('pincode')
        this.checkPincodeAvailability()
    }

    disableAddButton(disable) {
        document.querySelector('variant-radios')?.toggleAddButton(disable, 'Add pincode')
        document.querySelector('variant-selects')?.toggleAddButton(disable, 'Add pincode')

        document.querySelectorAll('form[action="/cart/add"] button').forEach( button => {
            if (disable)
                button.setAttribute("title", "Add a pincode to continue!")
            else
                button.removeAttribute("title")
        })
    }
}

customElements.define('pincode-checker', PincodeChecker);
```
4. **Now we are ready to this in PDP - Paste this in Schema**
```json
    {
      "type": "pincode",
      "name": "Pincode checker",
      "limit": 1,
      "settings": [
        {
          "type": "inline_richtext",
          "id": "heading",
          "default": "Check availability at",
          "label": "t:sections.image-with-text.blocks.heading.settings.heading.label"
        },
        {
          "type": "select",
          "id": "heading_size",
          "options": [
            {
              "value": "h2",
              "label": "t:sections.all.heading_size.options__1.label"
            },
            {
              "value": "h1",
              "label": "t:sections.all.heading_size.options__2.label"
            },
            {
              "value": "h0",
              "label": "t:sections.all.heading_size.options__3.label"
            }
          ],
          "default": "h1",
          "label": "t:sections.all.heading_size.label"
        },
        {
          "type": "text",
          "id": "placeholder_text",
          "default": "Enter Pincode",
          "label": "Placeholder text"
        },
        {
          "type": "text",
          "id": "pincode_available_message",
          "default": "We deliver to your pincode.",
          "label": "Pincode available message"
        },
        {
          "type": "text",
          "id": "pincode_unavailable_message",
          "default": "Sorry, Pincode not serviceable!",
          "label": "Pincode unavailable message"
        },
        {
          "type": "text",
          "id": "button_text",
          "default": "Check",
          "label": "Button text"
        }
      ]
    }
```
5. **Render the above snippet in the "main-product.liquid" file anywhere above the "buy-buttons**
   ```html
    {%- when 'pincode' -%}
      {% render 'pincode-checker',
        block: block
      %}
   ```

