<!--
 Copyright (c) 2021 - for information on the respective copyright owner
 see the NOTICE file and/or the repository at
 https://github.com/boschresearch/gx-ssi-demonstrator

 SPDX-License-Identifier: Apache-2.0
-->
<template>
  <div>
    <h1>Issue Organization Credential</h1>
    <v-text-field
      v-model="legalName"
      hint="The legalName that is going to be issued"
      label="Organization Name"
    />
    <v-text-field
      v-model="holder"
      hint="e.g. http://localhost/user1"
      label="Holder / Receiver / Subject ID"
    />
    <v-btn @click="createVC()">
      Create
    </v-btn>
    <br><br>
    <v-btn v-if="credential" alt small @click="copyToClipboard()">
      <v-icon>mdi-content-copy</v-icon>
    </v-btn><pre>{{ credential }}</pre>
  </div>
</template>

<script>
import vc from 'vc-js'
import jsonld from 'jsonld'
import { Ed25519VerificationKey2018 } from '@digitalbazaar/ed25519-verification-key-2018'
import { Ed25519Signature2018 } from '@digitalbazaar/ed25519-signature-2018'

export default {
  data () {
    return {
      legalName: '',
      holder: '',
      credential: null
    }
  },
  methods: {
    async createVC () {
      try {
        const credential = {
          '@context': [
            'https://www.w3.org/2018/credentials/v1',
            'https://schema.org/docs/jsonldcontext.jsonld',
            {
              '@version': 1.1,
              '@protected': true,
              id4me: 'https://id4me.org/definitions#',
              testing: {
                '@id': 'id4me:testing',
                '@type': 'xsd:string'
              },
              IDAuthorityCredential: 'id4me:IDAuthorityCredential',
              isIDAuthority: {
                '@id': 'id4me:isIDAuthority',
                '@context': {
                  '@protected': true,
                  operationalCountry: {
                    '@id': 'id4me:operationalCountry',
                    '@type': 'http://schema.org/addressCountry'
                  },
                  operationalJurisdiction: {
                    '@id': 'id4me:operationalJurisdiction',
                    '@type': 'http://schema.org/addressCountry'
                  },
                  privacyFrameworks: {
                    '@id': 'id4me:privacyFrameworks',
                    '@type': 'id4me:privacyFrameworks'
                  },
                  operatorType: {
                    '@id': 'id4me:operatorType',
                    '@type': 'id4me:operatorType'
                  },
                  trustLevel: {
                    '@id': 'id4me:trustLevel',
                    '@type': 'id4me:trustLevel'
                  }
                }
              }
            }
          ],
          id: new Date().getTime().toString(),
          type: ['VerifiableCredential', 'IDAuthorityCredential'],
          issuer: this.$store.state.keyPair.controller,
          issuanceDate: new Date().toISOString(),
          credentialSubject: {
            type: 'Organization',
            name: this.legalName,
            legalName: this.legalName,
            address: {
              type: 'PostalAddress',
              streetAddress: '123 Main St.',
              addressLocality: 'Berlin',
              postalCode: '24060',
              addressCountry: 'DE'
            },
            isIDAuthority: {
              operatorType: 'IDAuthority',
              operationalCountry: 'DE',
              operationalJurisdiction: 'DE',
              privacyFrameworks: [
                'gdpr'
              ],
              trustLevel: 'id4me_otl_member'
            }
          }
        }
        console.log('credential', credential)

        const keypairObj = this.$store.state.keyPair
        console.log('keypairObj', keypairObj)
        const keyPair = await Ed25519VerificationKey2018.from(keypairObj)
        const suite = new Ed25519Signature2018({
          verificationMethod: keypairObj.id,
          controller: keypairObj.controller,
          key: keyPair
        })
        console.log('suite', suite)

        const verifiableCredential = await vc.issue({
          credential,
          suite,
          compactProof: false, // security-v2 issues
          documentLoader: jsonld.documentLoaders.xhr()
        })
        console.log('verifiableCredential', verifiableCredential)
        this.credential = verifiableCredential
      } catch (e) {
        console.log('catch', JSON.stringify(e, null, 4))
        console.log('catch2', e)
      }
    },
    copyToClipboard () {
      navigator.clipboard.writeText(JSON.stringify(this.credential, null, 4))
    }
  }

}
</script>

<style>

</style>
