import java.io.*;
import java.nio.file.*;
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.util.*;

/**
 * Sistema de cadastro e notificação de eventos (Console)
 */
public class Main {
    public static void main(String[] args) {
        new App().run();
    }
}

/** Controla o fluxo da aplicação e o menu interativo de console. */
class App {
    private final Scanner scanner = new Scanner(System.in);
    private final EventService eventService = new EventService("events.data");
    private final ParticipationService participationService = new ParticipationService();
    private final NotificationService notificationService = new NotificationService(eventService);
    private final UserService userService = new UserService("current_user.data"); // Persistência de usuário

    private User currentUser() { return userService.getCurrentUser(); }

    void run() {
        eventService.loadFromDisk();
        showWelcome();
        mainLoop();
    }

    private void showWelcome() {
        System.out.println("========================================");
        System.out.println("  SISTEMA DE EVENTOS - CONSOLE (Java)");
        System.out.println("========================================\n");
        System.out.println("Dica: Datas no formato  dd/MM/yyyy HH:mm  (ex.: 10/09/2025 19:30)\n");
        if (currentUser() != null) {
            System.out.println("Usuário carregado: " + currentUser().getName() + "\n");
        }
    }

    private void ensureUserIsRegistered() {
        if (currentUser() != null) return;
        System.out.println("Nenhum usuário logado. Cadastre-se para continuar.\n");
        User u = createUser();
        userService.setCurrentUser(u);
        System.out.println("\nBem-vindo(a), " + u.getFirstName() + "!\n");
    }

    private void mainLoop() {
        while (true) {
            printMainMenu();
            String opt = readLine("Escolha uma opção: ");
            switch (opt) {
                case "1":
                    User u = createUser();
                    userService.setCurrentUser(u);
                    break;
                case "2":
                    ensureUserIsRegistered();
                    createEventFlow();
                    break;
                case "3":
                    listAllEventsOrdered();
                    break;
                case "4":
                    ensureUserIsRegistered();
                    participateInEvent();
                    break;
                case "5":
                    ensureUserIsRegistered();
                    listMyParticipations();
                    break;
                case "6":
                    ensureUserIsRegistered();
                    cancelParticipation();
                    break;
                case "7":
                    showHappeningNow();
                    break;
                case "8":
                    showPastEvents();
                    break;
                case "9":
                    listCategories();
                    break;
                case "0":
                    System.out.println("Salvando e saindo...");
                    eventService.saveToDisk();
                    userService.saveToDisk();
                    return;
                default:
                    System.out.println("Opção inválida.\n");
            }
        }
    }

    private void printMainMenu() {
        System.out.println("---------------- MENU ----------------");
        System.out.println("1) Cadastrar/Alterar Usuário");
        System.out.println("2) Cadastrar Evento");
        System.out.println("3) Listar Eventos (ordenados por data/hora)");
        System.out.println("4) Participar de um Evento");
        System.out.println("5) Meus Eventos Confirmados");
        System.out.println("6) Cancelar Participação");
        System.out.println("7) Eventos ocorrendo AGORA");
        System.out.println("8) Eventos que JÁ ocorreram");
        System.out.println("9) Ver Categorias Disponíveis");
        System.out.println("0) Sair");
        System.out.println("--------------------------------------");
    }

    private User createUser() {
        System.out.println("--- Cadastro de Usuário ---");
        String name = readLine("Nome completo: ");
        String email = readLine("Email: ");
        String city = readLine("Cidade: ");
        String phone = readLine("Telefone (opcional): ");
        return new User(name, email, city, phone);
    }

    private void listCategories() {
        System.out.println("Categorias de eventos:");
        for (EventCategory c : EventCategory.values()) {
            System.out.println("- " + c);
        }
        System.out.println();
    }

    private void createEventFlow() {
        System.out.println("--- Cadastro de Evento ---");
        String name = readLine("Nome: ");
        String address = readLine("Endereço: ");
        EventCategory category = askCategory();
        LocalDateTime when = askDateTime("Data e hora (dd/MM/yyyy HH:mm): ");
        String description = readLine("Descrição: ");
        Event e = new Event(UUID.randomUUID().toString(), name, address, category, when, description);
        eventService.addEvent(e);
        System.out.println("Evento cadastrado com sucesso!\n");
        notificationService.notifyIfUpcoming(e);
    }

    private void listAllEventsOrdered() {
        System.out.println("--- Eventos (ordenados) ---");
        List<Event> ordered = eventService.getAllOrdered();
        if (ordered.isEmpty()) {
            System.out.println("Nenhum evento cadastrado.\n");
            return;
        }
        int i = 1;
        for (Event e : ordered) {
            System.out.printf("%d) %s\n", i++, renderEventLine(e));
        }
        System.out.println();
    }

    private void participateInEvent() {
        List<Event> ordered = eventService.getAllOrdered();
        if (ordered.isEmpty()) {
            System.out.println("Nenhum evento disponível.\n");
            return;
        }
        listAllEventsOrdered();
        int idx = readInt("Digite o número do evento para participar: ");
        if (idx < 1 || idx > ordered.size()) {
            System.out.println("Índice inválido.\n");
            return;
        }
        Event chosen = ordered.get(idx - 1);
        participationService.confirmParticipation(currentUser(), chosen);
        System.out.println("Participação confirmada em: " + chosen.getName() + "\n");
    }

    private void listMyParticipations() {
        System.out.println("--- Meus eventos confirmados ---");
        List<Event> events = participationService.eventsOf(currentUser());
        if (events.isEmpty()) {
            System.out.println("Você ainda não confirmou presença em nenhum evento.\n");
            return;
        }
        int i = 1;
        for (Event e : events.stream().sorted(Comparator.comparing(Event::getWhen)).toList()) {
            System.out.printf("%d) %s\n", i++, renderEventLine(e));
        }
        System.out.println();
    }

    private void cancelParticipation() {
        List<Event> my = participationService.eventsOf(currentUser());
        if (my.isEmpty()) {
            System.out.println("Você não possui participações cadastradas.\n");
            return;
        }
        System.out.println("--- Cancelar participação ---");
        int i = 1;
        for (Event e : my.stream().sorted(Comparator.comparing(Event::getWhen)).toList()) {
            System.out.printf("%d) %s\n", i++, renderEventLine(e));
        }
        int idx = readInt("Digite o número do evento para cancelar: ");
        if (idx < 1 || idx > my.size()) {
            System.out.println("Índice inválido.\n");
            return;
        }
        Event chosen = my.get(idx - 1);
        participationService.cancelParticipation(currentUser(), chosen);
        System.out.println("Participação cancelada em: " + chosen.getName() + "\n");
    }

    private void showHappeningNow() {
        System.out.println("--- Eventos ocorrendo AGORA ---");
        List<Event> now = eventService.happeningNow();
        if (now.isEmpty()) {
            System.out.println("Nenhum evento ocorrendo neste momento.\n");
            return;
        }
        for (Event e : now) {
            System.out.println("- " + renderEventLine(e));
        }
        System.out.println();
    }

    private void showPastEvents() {
        System.out.println("--- Eventos que já ocorreram ---");
        List<Event> past = eventService.pastEvents();
        if (past.isEmpty()) {
            System.out.println("Nenhum evento passado registrado.\n");
            return;
        }
        for (Event e : past) {
            System.out.println("- " + renderEventLine(e));
        }
        System.out.println();
    }

    private String renderEventLine(Event e) {
        String whenStr = e.getWhen().format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm"));
        String status = eventService.statusOf(e);
        return String.format("%s | %s | %s | %s | %s", e.getName(), e.getAddress(), e.getCategory(), whenStr, status);
    }

    private EventCategory askCategory() {
        while (true) {
            System.out.println("Escolha a categoria:");
            int i = 1;
            for (EventCategory c : EventCategory.values()) {
                System.out.println(i + ") " + c);
                i++;
            }
            int opt = readInt("Número da categoria: ");
            if (opt >= 1 && opt <= EventCategory.values().length) {
                return EventCategory.values()[opt - 1];
            }
            System.out.println("Opção inválida. Tente novamente.\n");
        }
    }

    private LocalDateTime askDateTime(String prompt) {
        DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
        while (true) {
            String s = readLine(prompt);
            try {
                return LocalDateTime.parse(s, fmt);
            } catch (DateTimeParseException e) {
                System.out.println("Formato inválido. Use dd/MM/yyyy HH:mm.\n");
            }
        }
    }

    private String readLine(String prompt) {
        System.out.print(prompt);
        return scanner.nextLine().trim();
    }

    private int readInt(String prompt) {
        while (true) {
            String s = readLine(prompt);
            try {
                return Integer.parseInt(s);
            } catch (NumberFormatException e) {
                System.out.println("Digite um número válido.\n");
            }
        }
    }
}

/** Entidade de domínio: Usuário do sistema. */
class User {
    private final String name;
    private final String email;
    private final String city;
    private final String phone;

    public User(String name, String email, String city, String phone) {
        this.name = name;
        this.email = email;
        this.city = city;
        this.phone = phone;
    }

    public String getName() { return name; }
    public String getFirstName() {
        String[] parts = name.trim().split("\\s+");
        return parts.length > 0 ? parts[0] : name;
    }
    public String getEmail() { return email; }
    public String getCity() { return city; }
    public String getPhone() { return phone; }
}

/** Serviço de persistência de usuário. */
class UserService {
    private User currentUser = null;
    private final Path filePath;

    public UserService(String fileName) {
        this.filePath = Paths.get(fileName);
        loadFromDisk();
    }

    public User getCurrentUser() { return currentUser; }

    public void setCurrentUser(User user) {
        this.currentUser = user;
        saveToDisk();
    }

    public void loadFromDisk() {
        if (!Files.exists(filePath)) return;
        try {
            List<String> lines = Files.readAllLines(filePath);
            if (!lines.isEmpty()) {
                String[] parts = lines.get(0).split("\\|", -1);
                if (parts.length >= 4) {
                    currentUser = new User(parts[0], parts[1], parts[2], parts[3]);
                }
            }
        } catch (IOException e) {
            System.err.println("Falha ao ler usuário: " + e.getMessage());
        }
    }

    public void saveToDisk() {
        if (currentUser == null) return;
        try (BufferedWriter bw = Files.newBufferedWriter(filePath)) {
            bw.write(String.join("|",
                    currentUser.getName(),
                    currentUser.getEmail(),
                    currentUser.getCity(),
                    currentUser.getPhone() != null ? currentUser.getPhone() : ""
            ));
        } catch (IOException e) {
            System.err.println("Falha ao salvar usuário: " + e.getMessage());
        }
    }
}

/** Categorias pré-definidas para eventos. */
enum EventCategory {
    FESTA, ESPORTIVO, SHOW, TEATRO, EDUCACAO, TECNOLOGIA, RELIGIOSO, OUTROS
}

/** Entidade de domínio: Evento. */
class Event {
    private final String id;
    private final String name;
    private final String address;
    private final EventCategory category;
    private final LocalDateTime when;
    private final String description;

    public Event(String id, String name, String address, EventCategory category, LocalDateTime when, String description) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.category = category;
        this.when = when;
        this.description = description;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public String getAddress() { return address; }
    public EventCategory getCategory() { return category; }
    public LocalDateTime getWhen() { return when; }
    public String getDescription() { return description; }
}

/** Serviço de persistência e consulta de eventos. */
class EventService {
    private static final Duration DEFAULT_DURATION = Duration.ofHours(2);
    private final List<Event> events = new ArrayList<>();
    private final Path filePath;
    private final DateTimeFormatter ISO_FMT = DateTimeFormatter.ISO_LOCAL_DATE_TIME;

    public EventService(String fileName) { this.filePath = Paths.get(fileName); }

    public void addEvent(Event e) {
        events.add(e);
        events.sort(Comparator.comparing(Event::getWhen));
        saveToDisk();
    }

    public List<Event> getAllOrdered() {
        return events.stream().sorted(Comparator.comparing(Event::getWhen)).toList();
    }

    public List<Event> happeningNow() {
        LocalDateTime now = LocalDateTime.now();
        return events.stream().filter(e -> isHappeningNow(e, now)).toList();
    }

    public List<Event> pastEvents() {
        LocalDateTime now = LocalDateTime.now();
        return events.stream().filter(e -> e.getWhen().isBefore(now.minus(DEFAULT_DURATION))).sorted(Comparator.comparing(Event::getWhen)).toList();
    }

    public String statusOf(Event e) {
        LocalDateTime now = LocalDateTime.now();
        if (isHappeningNow(e, now)) return "OCORRENDO AGORA";
        if (e.getWhen().isBefore(now)) return "JÁ OCORREU";
        long minutes = Duration.between(now, e.getWhen()).toMinutes();
        if (minutes <= 0) minutes = 0;
        return String.format("em %d min", minutes);
    }

    private boolean isHappeningNow(Event e, LocalDateTime ref) {
        LocalDateTime start = e.getWhen();
        LocalDateTime end = start.plus(DEFAULT_DURATION);
        return !ref.isBefore(start) && !ref.isAfter(end);
    }

    public void loadFromDisk() {
        events.clear();
        if (!Files.exists(filePath)) return;
        try (BufferedReader br = Files.newBufferedReader(filePath)) {
            String line;
            while ((line = br.readLine()) != null) {
                if (line.isBlank()) continue;
                Optional<Event> e = parseLine(line);
                e.ifPresent(events::add);
            }
            events.sort(Comparator.comparing(Event::getWhen));
        } catch (IOException ex) {
            System.err.println("Falha ao ler arquivo: " + ex.getMessage());
        }
    }

    public void saveToDisk() {
        try (BufferedWriter bw = Files.newBufferedWriter(filePath)) {
            for (Event e : events) {
                bw.write(serialize(e));
                bw.newLine();
            }
        } catch (IOException ex) {
            System.err.println("Falha ao salvar arquivo: " + ex.getMessage());
        }
    }

    private Optional<Event> parseLine(String line) {
        try {
            String[] parts = line.split("\\|", -1);
            if (parts.length < 6) return Optional.empty();
            String id = parts[0];
            String name = unescape(parts[1]);
            String address = unescape(parts[2]);
            EventCategory category = EventCategory.valueOf(parts[3]);
            LocalDateTime when = LocalDateTime.parse(parts[4], ISO_FMT);
            String description = unescape(parts[5]);
            return Optional.of(new Event(id, name, address, category, when, description));
        } catch (Exception e) {
            System.err.println("Linha inválida em events.data: " + line);
            return Optional.empty();
        }
    }

    private String serialize(Event e) {
        return String.join("|",
                e.getId(),
                escape(e.getName()),
                escape(e.getAddress()),
                e.getCategory().name(),
                e.getWhen().format(ISO_FMT),
                escape(e.getDescription())
        );
    }

    private String escape(String s) { return s.replace("|", "/").replace("\n", " ").trim(); }
    private String unescape(String s) { return s; }
}

/** Serviço simples de participação (em memória). */
class ParticipationService {
    private final Map<String, Set<String>> participations = new HashMap<>();

    public void confirmParticipation(User u, Event e) {
        participations.computeIfAbsent(u.getEmail(), k -> new HashSet<>()).add(e.getId());
    }

    public void cancelParticipation(User u, Event e) {
        Set<String> set = participations.get(u.getEmail());
        if (set != null) set.remove(e.getId());
    }

    public List<Event> eventsOf(User u) {
        Set<String> ids = participations.getOrDefault(u.getEmail(), Collections.emptySet());
        return AppUtils.filterEventsByIds(AppUtils.getEventServiceFromContext(), ids);
    }
}

/** Serviço de notificações simples (console). */
class NotificationService {
    public NotificationService(EventService eventService) { AppUtils.registerEventService(eventService); }

    public void notifyIfUpcoming(Event e) {
        long minutes = Duration.between(LocalDateTime.now(), e.getWhen()).toMinutes();
        if (minutes >= 0 && minutes <= 60) {
            System.out.println("[Notificação] Evento próximo: " + e.getName() + " em " + minutes + " min.");
        }
    }
}

/** Utilitário para compartilhar o EventService com ParticipationService */
class AppUtils {
    private static EventService _eventService;

    static void registerEventService(EventService s) { _eventService = s; }
    static EventService getEventServiceFromContext() { return _eventService; }

    static List<Event> filterEventsByIds(EventService s, Set<String> ids) {
        if (s == null || ids == null || ids.isEmpty()) return List.of();
        List<Event> out = new ArrayList<>();
        for (Event e : s.getAllOrdered()) {
            if (ids.contains(e.getId())) out.add(e);
        }
        return out;
    }
}
