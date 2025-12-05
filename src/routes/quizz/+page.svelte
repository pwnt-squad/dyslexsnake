<script>
  // Questions
  const questions = [
    {
      id: "usage",
      text: "Certaines distributions sont conçues pour un usage spécifique, d'autres pour un usage quotidien. De quoi avez-vous besoin ?",
      multiple: true,
      answers: [
        {value: 0, text: "Je veux une distribution pour un usage quotidien."},
        {value: 1, text: "Je veux utiliser Linux pour naviguer anonymement sur le web."},
        {value: 2, text: "Je préfère une distribution qui est prise en charge par les éditeurs de jeux."},
        {value: 3, text: "Je veux exécuter tous les programmes dans une zone isolée."},
        {value: 4, text: "L'utilisateur est malvoyant."},
        {value: 5, text: "Dispose d'un mode « live » : Peut être démarré à l'aide d'un disque ou d'une clé USB pour la tester (avant l'installation)"}
      ]
    },
    {
      id: "pcknowledge",
      text: "Comment évaluez-vous vos compétences en matière d'utilisation d'un ordinateur ?",
      answers: [
        {value: 0, text: "J'ai souvent besoin de l'aide des autres pour régler des problèmes avec mon ordinateur"},
        {value: 1, text: "Je suis déjà capable de résoudre certains problèmes informatiques par moi-même"},
        {value: 2, text: "Je peux résoudre la plupart ou la totalité des problèmes informatiques par moi-même"}
      ]
    },
    {
      id: "linuxknowledge",
      text: "Comment évaluez-vous votre connaissance de Linux lui-même ?",
      answers: [
        {value: 0, text: "J'ai peu ou pas de connaissances sur Linux"},
        {value: 1, text: "J'ai déjà utilisé Linux pour diverses raisons"},
        {value: 2, text: "J'ai une bonne compréhension du système d'exploitation Linux"}
      ]
    },
    {
      id: "installation",
      text: "Combien de paramètres souhaitez-vous modifier par vous-même lors de l'installation ? Combien doivent-ils être prédéfinis ?",
      multiple: true,
      answers: [
        {value: 0, text: "Je veux utiliser les valeurs par défaut de l'assistant d'installation"},
        {value: 1, text: "Je veux choisir les paramètres par moi-même"},
        {value: 2, text: "Je veux configurer autant de paramètres que possible à travers des applications graphiques"}
      ]
    },
    {
      id: "hardware",
      text: "Il est important de connaître l'âge de votre matériel informatique, car certaines distributions ne fonctionneront pas sur de vieux ordinateurs.",
      answers: [
        {value: 0, text: "Je ne suis pas sûr et/ou mon ordinateur a déjà des problèmes de performance et/ou ne supporte que le 32 bits"},
        {value: 1, text: "Les performances de mon ordinateur sont bonnes, je n'ai aucun problème de performances, mon matériel supporte le 64 bits"},
      ]
    },
    {
      id: "help",
      text: "Certaines distributions privilégient les articles dans les wikis pour trouver les solutions aux problèmes. Que préférez-vous ?",
      answers: [
        {value: 0, text: "Je suis capable de résoudre les problèmes que je rencontre en utilisant les guides des pages du wiki ou des tutoriels."},
        {value: 1, text: "Je préfère demander de l'aide aux autres"},
      ]
    },
    {
      id: "ux",
      text: "Linux peut utiliser différentes interface graphiques (« Environnements de Bureau »). La majorité des distributions en fournissent un par défaut. Il est important de savoir si vous avez une préférence sachant que vous pouvez en changer à n’importe quel moment !",
      answers: [
        {value: 0, text: "Je préfère une interface utilisateur de type Windows"},
        {value: 1, text: "Je préfère une interface utilisateur de type macOS"},
      ]
    },
    {
      id: "price",
      text: "La plupart des distributions sont gratuites. Certaines distributions offrent un support supplémentaire en échange d'une contribution unique.",
      answers: [
        {value: 0, text: "Je préfère une distribution gratuite"},
        {value: 1, text: "Je suis prêt à payer une contribution"},
      ]
    },
    {
      id: "scope",
      text: "Les diverses distributions préinstallent différents logiciels. Certaines installent une suite complète de programmes de base pour que vous ayez tout à portée de main. D'autres distributions ne proposent qu'une installation de base pour laisser ensuite l'utilisateur installer les programmes qu'il souhaite utiliser.",
      answers: [
        {value: 0, text: "Je préfère une distribution embarquant tous les programmes de base dont j'ai besoin pour commencer à travailler immédiatement après l'installation"},
        {value: 1, text: "Je veux choisir les programmes de base à installer par moi-même"},
      ]
    },
    {
      id: "ideology",
      text: "L'idéologie (de licence) d'une distribution est un débat controversé. Il existe des distributions qui utilisent principalement des « licences libres », d'autres utilisent également des logiciels « non libres ».",
      answers: [
        {value: 0, text: "Je veux utiliser des licences libres autant que possible"},
        {value: 1, text: "Je suis prêt à accepter des licences non libres tant que mon système fonctionne"},
      ]
    },
    {
      id: "privacy",
      text: "Certaines distributions utilisent des services en ligne pour améliorer l'expérience utilisateur. Cela peut affecter la vie privée de l'utilisateur, c'est-à-dire que celui-ci peut être suivi lorsqu'il utilise ces services en ligne.",
      answers: [
        {value: 0, text: "Je ne veux pas de ça"},
        {value: 1, text: "Si cela améliore mon expérience utilisateur, je suis favorable à l'utilisation de ces services en ligne"},
      ]
    },
    {
      id: "administration",
      text: "Il existe de nombreuses façons d'administrer une distribution Linux, par exemple pour installer des logiciels.",
      multiple: true,
      answers: [
        {value: 0, text: "Je veux installer mes programmes en utilisant une sorte d'« App store »."},
        {value: 1, text: "Je veux installer des logiciels en utilisant principalement des commandes de terminal"},
        {value: 2, text: "Je veux éviter d'utiliser systemd"},
      ]
    },
    {
      id: "updates",
      text: "Certaines distributions offrent des mises à jour plus rapidement, ce qui peut affecter la stabilité du système. Que préférez-vous ?",
      answers: [
        {value: 0, text: "Je préfère obtenir des mises à jour rapidement"},
        {value: 1, text: "Je préfère des mises à jour stables"},
      ]
    }
  ];

  const distros = [
    {
      id: "ubuntu",
      name: "Ubuntu",
      attrs: {
        usage: [0,5],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0,1],
        ideology: [0,1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "linuxmint",
      name: "Linux Mint",
      attrs: {
        usage: [0,5],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "zorin",
      name: "Zorin OS",
      attrs: {
        usage: [0],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "elementary",
      name: "elementary OS",
      attrs: {
        usage: [0],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [1],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "lubuntu",
      name: "Lubuntu",
      attrs: {
        usage: [0,5],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [0,1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "kubuntu",
      name: "Kubuntu",
      attrs: {
        usage: [0],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "xubuntu",
      name: "Xubuntu",
      attrs: {
        usage: [0,5],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [0,1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "ubuntu-mate",
      name: "Ubuntu MATE",
      attrs: {
        usage: [0],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0,1]
      }
    },
    {
      id: "popos",
      name: "Pop!_OS",
      attrs: {
        usage: [0,2],
        pcknowledge: 1,
        linuxknowledge: [1],
        installation: [1,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [0]
      }
    },
    {
      id: "linux-mx",
      name: "MX Linux",
      attrs: {
        usage: [0,5],
        pcknowledge: 0,
        linuxknowledge: [0,1],
        installation: [0,2],
        hardware: [1],
        help: [0,1],
        ux: [0],
        scope: [0,1],
        ideology: [1],
        privacy: [1],
        administration: [0,1],
        updates: [1]
      }
    },
    { id: "debian", name: "Debian", attrs: { usage: [0,5], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[0,1], updates:[1] } },
    { id: "devuan", name: "Devuan", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0], privacy:[1], administration:[1,2], updates:[1] } },
    { id: "artix", name: "Artix", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0], privacy:[1], administration:[1,2], updates:[0] } },
    { id: "gentoo", name: "Gentoo", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0], privacy:[1], administration:[1,2], updates:[0] } },
    { id: "arch", name: "Arch Linux", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0], privacy:[1], administration:[1,2], updates:[0] } },
    { id: "crux", name: "Crux Linux", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0], privacy:[1], administration:[1,2], updates:[0] } },
    { id: "nixos", name: "NixOS", attrs: { usage:[2], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[1,2], updates:[0] } },
    { id: "fedora", name: "Fedora", attrs: { usage:[0,2], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[0,1], updates:[0] } },
    { id: "opensuse", name: "openSUSE", attrs: { usage:[0,2], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[0,1], updates:[0,1] } },
    { id: "rhel", name: "Red Hat Enterprise Linux", attrs: { usage:[0,2], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[0,1], updates:[1] } },
    { id: "rocky", name: "Rocky Linux", attrs: { usage:[0,2], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[1], ideology:[0,1], privacy:[1], administration:[0,1], updates:[1] } },
    { id: "pclinuxos", name: "PCLinuxOS", attrs: { usage:[0], pcknowledge:0, linuxknowledge:[0,1], installation:[0,2], hardware:[1], help:[0,1], ux:[0], scope:[0], ideology:[1], privacy:[1], administration:[0,1], updates:[0,1] } },
    { id: "solus", name: "Solus", attrs: { usage:[0], pcknowledge:0, linuxknowledge:[0,1], installation:[0,2], hardware:[1], help:[0,1], ux:[0], scope:[0], ideology:[1], privacy:[1], administration:[0,1], updates:[0,1] } },
    { id: "manjaro", name: "Manjaro", attrs: { usage:[0,2], pcknowledge:1, linuxknowledge:[1,2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[0,1], ideology:[1], privacy:[1], administration:[0,1], updates:[0] } },
    { id: "deepin", name: "Deepin", attrs: { usage:[0], pcknowledge:0, linuxknowledge:[0,1], installation:[0,2], hardware:[1], help:[0,1], ux:[1], scope:[0], ideology:[1], privacy:[1], administration:[0,1], updates:[0,1] } },
    { id: "tails", name: "Tails", attrs: { usage:[1,5], pcknowledge:1, linuxknowledge:[1], installation:[0,2], hardware:[1], help:[0,1], ux:[0], scope:[0], ideology:[0,1], privacy:[0], administration:[0], updates:[0] } },
    { id: "qubes", name: "Qubes OS", attrs: { usage:[3], pcknowledge:2, linuxknowledge:[2], installation:[1,2], hardware:[1], help:[0,1], ux:[0], scope:[0,1], ideology:[0,1], privacy:[0], administration:[1,2], updates:[0] } },
    { id: "knoppix", name: "Knoppix", attrs: { usage:[5], pcknowledge:0, linuxknowledge:[0,1], installation:[0], hardware:[1], help:[0,1], ux:[0], scope:[0], ideology:[1], privacy:[1], administration:[0,1], updates:[0,1] } }
  ];

  let step = 0;
  let answers = {};

  function selectAnswer(answer) {
    const questionId = questions[step].id;
    if (questions[step].multiple) {
      // Handle multiple answers (checkboxes)
      if (answers[questionId]?.includes(answer.value)) {
        answers[questionId] = answers[questionId].filter(a => a !== answer.value); // Deselect
      } else {
        answers[questionId] = [...(answers[questionId] || []), answer.value]; // Select
      }
    } else {
      // Handle single answer (radio buttons)
      answers[questionId] = answer.value;
    }
  }

  function evaluate(distro) {
    let msgs = [];

    // Personnalisation selon les connaissances en informatique
    const qPcknowledge = answers.pcknowledge;
    if (qPcknowledge !== undefined) {
        if (qPcknowledge < distro.attrs.pcknowledge) {
            msgs.push(`- ${distro.name} nécessite plus de connaissances informatiques que celles que vous avez spécifiées.`);
        }
    }

    // Personnalisation selon les connaissances Linux
    const qLinuxKnowledge = answers.linuxknowledge;
    if (qLinuxKnowledge !== undefined) {
        if (qLinuxKnowledge < distro.attrs.linuxknowledge) {
            msgs.push(`- ${distro.name} est probablement trop avancée si vous n'avez pas d'expérience avec Linux.`);
        }
    }

    // Personnalisation selon l'usage
    const qUsage = answers.usage; // could be number or array
    if (qUsage !== undefined) {
        const selected = Array.isArray(qUsage) ? qUsage : [qUsage];
        selected.forEach(ans => {
            if (ans === 0) {
                msgs.push(
                    distro.attrs.usage.includes(0)
                        ? `+ Utilisation adaptée pour un usage quotidien`
                        : `- Une utilisation quotidienne peut entraîner des difficultés majeures`
                );
            }
            if (ans === 1) {
                msgs.push(
                    distro.attrs.usage.includes(1)
                        ? `+ Conçu pour des fins d'anonymisation`
                        : `- Non conçu pour des fins d'anonymisation`
                );
            }
            if (ans === 2) {
                msgs.push(
                    distro.attrs.usage.includes(2)
                        ? `+ Bonne prise en charge par les éditeurs de jeu `
                        : `- Mauvaise prise en charge par les éditeurs de jeu `
                );
            }
            if (ans === 3) {
                msgs.push(
                    distro.attrs.usage.includes(3)
                        ? `+ Execute les programmes dans une zone isolée`
                        : `- N'exécute pas les programmes dans une zone isolée`
                );
            }
            if (ans === 4) {
                msgs.push(
                    distro.attrs.usage.includes(4)
                        ? `+ Accessibilité pour les malvoyants`
                        : `- Pas de mesures d'accessibilité pour les malvoyants`
                );
            }
            if (ans === 5) {
                msgs.push(
                    distro.attrs.usage.includes(5)
                        ? `+ Dispose d'un mode "live"`
                        : `- Ne dispose pas d'un mode "live"`
                );
            }

        });
    }

    // Personnalisation selon l'installation
    const qInstallation = answers.installation; // could be number or array
    if (qInstallation !== undefined) {
        const selected = Array.isArray(qInstallation) ? qInstallation : [qInstallation];
        selected.forEach(ans => {
            if (ans === 0) {
                msgs.push(
                    distro.attrs.installation.includes(0)
                        ? `+ Propose des valeurs par défaut lors de l'installation`
                        : `- Ne propose pas de valeur par défaut lors de l'installation`
                );
            }
            if (ans === 1) {
                msgs.push(
                    distro.attrs.installation.includes(1)
                        ? `+ Possible de choisir les paramètres d'installation soi-même`
                        : `- Ne propose pas de personnalisation des paramètres lors de l'installation`
                );
            }
            if (ans === 2) {
                msgs.push(
                    distro.attrs.installation.includes(2)
                        ? `+ Interface graphique pour la configuration des paramètres d'installation`
                        : `- Aucune interface graphique pour la configuration des paramètres lors de l'installation`
                );
            }
        });
    }

    // Personnalisation selon l'administration
    const qAdministration = answers.administration; // could be number or array
    if (qAdministration !== undefined) {
        const selected = Array.isArray(qInstallation) ? qInstallation : [qInstallation];
        selected.forEach(ans => {
            if (ans === 0) {
                msgs.push(
                    distro.attrs.installation.includes(0)
                        ? `+ L'installation de programme est possible avec un magasin d'application`
                        : `- Aucun magasin d'application disponible`
                );
            }
            if (ans === 1) {
                msgs.push(
                    distro.attrs.installation.includes(1)
                        ? `+ Gestion des applications principalement faite sur le terminal`
                        : `- La gestion des applications ne se fait pas principalement sur le terminal`
                );
            }
            if (ans === 2) {
                msgs.push(
                    distro.attrs.installation.includes(2)
                        ? `+ N'utilise pas systemd`
                        : `- Utilise systemd`
                );
            }
        });
    }



    // Personnalisation selon les mises à jour
    const qUpdates = answers.updates;
    if (qUpdates !== undefined) {
        if (qUpdates === 0) {
            msgs.push(
                distro.attrs.updates.includes(0)
                ? `+ Offre des mises à jour rapides.`
                : `- Privilégie des mises à jour stables et éprouvées.`
            );
        } else if (qUpdates === 1) {
            msgs.push(
                distro.attrs.updates.includes(1)
                ? `+ Privilégie des mises à jour stables et éprouvées.`
                : `- Offre des mises à jour rapides.`
            );
        }
    }

    // Personnalisation ideology
    const qIdeology = answers.ideology;
    if (qIdeology !== undefined) {
        if (qIdeology === 0) {
            msgs.push(
                distro.attrs.ideology.includes(0)
                ? `+ Privilégie les licences libres.`
                : `- Utilise des logiciels propriétaires.`
            );
        }
    }

    // Personnalisation privacy
    const qPrivacy = answers.privacy;
    if (qPrivacy !== undefined) {
        if (qPrivacy === 0) {
            msgs.push(
                distro.attrs.privacy.includes(0)
                ? `+ Ne recueille des données que lorsque des services en ligne sont utilisés`
                : `- Collecte des métriques propre à l'usage`
            );
        }
    }


    // Personnalisation help
    const qHelp = answers.help;
    if (qHelp !== undefined) {
        if (qHelp === 1) {
            msgs.push(
                distro.attrs.help.includes(1)
                ? `- Nécessite la lecture des manuels pour être utilisée`
                : `+ Il existe des tutoriels pour le dépannage et la résolution de problèmes`
            );
        }
    }

    // Personnalisation price
    // const qPrice = answers.price;
    // if (qPrice !== undefined) {
    //     if (qPrice === 0) {
    //         msgs.push(
    //             distro.attrs.price.includes(0)
    //             ? `+ Distribution gratuite`
    //             : `- Distribution `
    //         );
    // }

    return msgs;
  }

</script>

{#if step < questions.length}
  <div class="question">
    <h2>{questions[step].text}</h2>
    {#each questions[step].answers as a}
      {#if questions[step].multiple}
        <label>
          <input
            type="checkbox"
            value={a.value}
            on:change={() => selectAnswer(a)}
            checked={answers[questions[step].id]?.includes(a.value)}
          />
          {a.text}
        </label>
      {:else}
        <label>
          <input
            type="radio"
            name={questions[step].id}
            value={a.value}
            on:change={() => selectAnswer(a)}
            checked={answers[questions[step].id] === a.value}
          />
          {a.text}
        </label>
      {/if}
    {/each}
  </div>
  <div>
    <button on:click={() => step++}>Next question</button>
  </div>
  <div>
    <button on:click={() => step--}>Previous question</button>
  </div>
{:else}
  <h2>Résultats</h2>
  {#each distros as d}
    <div class="distro">
      <h3>{d.name}</h3>
      <ul>
        {#each evaluate(d) as msg}
          <li>{msg}</li>
        {/each}
      </ul>
    </div>
  {/each}
{/if}

<style>
  button { display: block; margin: 5px 0; padding: 8px 12px; }
  .distro { margin-top: 20px; border: 1px solid #ccc; padding: 10px; }
  label { display: block; margin: 5px 0; }
</style>

