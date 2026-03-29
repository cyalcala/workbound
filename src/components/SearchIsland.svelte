<script lang="ts">
  export let platforms: any[] = [];
  
  let query = '';
  
  // Fuzzy search reactivity
  $: filteredPlatforms = platforms.filter(p => {
    if (!query) return true;
    const lowerQuery = query.toLowerCase();
    return p.name.toLowerCase().includes(lowerQuery) || 
           p.category.toLowerCase().includes(lowerQuery) ||
           p.description.toLowerCase().includes(lowerQuery) ||
           p.tags.some((t: string) => t.toLowerCase().includes(lowerQuery));
  });

  // Strict sorting / taxonomy grouping requested by user (AI & Non-tech priority)
  $: groupedPlatforms = filteredPlatforms.reduce((acc, platform) => {
    const cat = platform.category;
    if (!acc[cat]) acc[cat] = [];
    acc[cat].push(platform);
    return acc;
  }, {} as Record<string, any[]>);

  // Hardcoded ordering for taxonomy UX hierarchy
  const categoryOrder = [
    "AI Training & Micro-Task",
    "Non-Tech / Support / VA",
    "Writing & Creative",
    "General Job Boards",
    "Tech & Development",
    "Resources & Meta-Directories"
  ];
</script>

<!-- The Search "Island" -->
<div class="mb-20 relative group max-w-4xl mx-auto">
  <div class="absolute inset-y-0 left-0 pl-6 flex items-center pointer-events-none">
    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-ink/30 group-focus-within:text-accent transition-colors duration-300" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
    </svg>
  </div>
  <input 
    type="text" 
    bind:value={query}
    placeholder="Search 'Data Annotation' or 'Customer Support'..."
    class="block w-full pl-20 pr-8 py-6 rounded-2xl border-2 border-transparent bg-white shadow-card text-ink placeholder-ink/30 focus:outline-none focus:border-accent/40 focus:ring-4 focus:ring-accent/10 transition-all duration-300 text-2xl font-medium tracking-tight"
  />
  
  {#if query && filteredPlatforms.length > 0}
    <div class="absolute right-6 top-1/2 -translate-y-1/2 text-sm font-bold text-accent/80 tracking-[0.1em] uppercase">
      {filteredPlatforms.length} Matches
    </div>
  {/if}
</div>

{#if filteredPlatforms.length === 0}
  <div class="text-center py-32">
    <h3 class="text-3xl font-semibold tracking-tight text-ink/60">No platforms found for "<span class="text-ink">{query}</span>"</h3>
    <p class="text-ink/40 mt-4 text-xl">Try adjusting your filters or search terms.</p>
  </div>
{:else}
  <!-- The Categorized Grid -->
  <div class="space-y-32">
    {#each categoryOrder as category}
      {#if groupedPlatforms[category] && groupedPlatforms[category].length > 0}
        <section>
          <div class="mb-10 flex items-baseline justify-between border-b-2 border-ink/5 pb-4">
            <h2 class="text-4xl md:text-5xl font-extrabold tracking-tighter text-ink flex items-center gap-4">
              <span class="w-3 h-8 bg-accent rounded-sm inline-block"></span>
              {category}
            </h2>
            <span class="text-sm font-bold tracking-widest text-ink/40 bg-ink/5 px-3 py-1.5 rounded-lg uppercase">{groupedPlatforms[category].length} Sites</span>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
            {#each groupedPlatforms[category] as platform (platform.id)}
              <a 
                href={platform.url} 
                target="_blank" 
                rel="noopener noreferrer" 
                class="group relative flex flex-col justify-between bg-white rounded-[1.5rem] p-8 shadow-sm border border-transparent hover:border-accent/40 hover:-translate-y-2 hover:shadow-xl transition-all duration-300 h-full"
              >
                <!-- Platform Header -->
                <div>
                  <div class="flex items-start justify-between mb-6">
                    <h3 class="text-2xl font-bold tracking-tight text-ink leading-tight group-hover:text-accent transition-colors duration-200">
                      {platform.name}
                    </h3>
                    {#if platform.aiExposure === 'Low'}
                      <span class="inline-flex items-center px-3 py-1 rounded-full text-[10px] font-bold tracking-widest uppercase bg-green-50 text-green-700 border border-green-200/50">
                        Human First
                      </span>
                    {:else if platform.aiExposure === 'Medium'}
                      <span class="inline-flex items-center px-3 py-1 rounded-full text-[10px] font-bold tracking-widest uppercase bg-yellow-50 text-yellow-700 border border-yellow-200/50">
                        AI Adj
                      </span>
                    {/if}
                  </div>
                  <p class="text-[1.05rem] text-ink/60 leading-relaxed font-medium">
                    {platform.description}
                  </p>
                </div>

                <!-- Tags / Pills -->
                <div class="mt-8 flex flex-wrap gap-2">
                  {#each platform.tags as tag}
                    <span class="inline-flex items-center px-3 py-1.5 rounded-lg text-xs font-bold bg-parchment text-ink/50 uppercase tracking-[0.1em] border border-ink/5">
                      {tag}
                    </span>
                  {/each}
                </div>
              </a>
            {/each}
          </div>
        </section>
      {/if}
    {/each}
  </div>
{/if}
